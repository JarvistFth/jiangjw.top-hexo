---
title: redis设计与实现读书笔记(7)--多机数据库之集群（cluster）
date: 2020-08-30 13:59:28
categories: redis
tags: [redis]
keywords: [redis]
---
redis多级数据库主要有三部分：1、主从模式；2、sentinel高可用解决方案；3、redis集群。
这是第三部分，redis cluster。集群通过分片的方式来进行数据共享，并且提供复制和故障转移的功能。

<!---more--->

## 节点
redis会根据配置确定是否开启集群模式，集群模式下redis会继续沿用单机模式下的一些组件，比如使用rdb和aof持久化、使用serverCron函数进行周期操作等等。

通过cluster meet可以申请向某ip和port申请握手。

集群模式下有自己的独特的集群数据结构。

### 集群数据结构
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200830145107.png)

clusterNode结构如上，保存了包括名字、创建节点的时间、槽数组等属性。

clusterNode还保存了clusterLink结构，这个结构是节点的连接信息（像webserver中的httpconnection）

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200830145907.png)

同时，每个节点还会保存一个clusterState结构，记录当前节点的视角下，集群的整体状态。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200830150036.png)

该结构包括指向自己的指针，配置纪元（用于故障转移）、集群的状态（上线还是下线）、集群节点dict（key是名字，值为一个clusterNode结构的指针）等。

具体的结构内容示例：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200830150459.png)


### cluster meet
MEET/PING/PONG三次握手。

## 槽指派
### 槽指派信息记录
redis cluster通过分片的方式保存数据库中的键值对，整个数据库被分成16384个slot，每个key都属于其中之一。每个node可以处理1-16384个slot。

当数据库中有一个槽没节点处理时，集群处于下线状态；

通过cluster addslots命令，可以将一个或多个槽委派该节点负责。

clusterNode的numslots属性和slots数组记录了节点负责哪些槽。

slots数组是个二进制位数组，长度为16384/8个字节，按照对应i位上的二进制位值来判断节点是否负责处理槽i。

比如i位上是1，就认为负责处理slot1.

### 传播节点的槽指派信息
节点会将自己的slots数组和numslot传播给其他节点，告诉其他节点自己正在处理哪些slot。然后其他节点收到信息，会更新其对应的clusterNode的slots数组和numslot。

### 记录集群中所有槽的指派信息
clusterState中slots数组记录了所有槽的指派信息，数组指向一个clusterNode结构，表示第i位的slot由该clusterNode处理。

这种方式处理是为了防止查找上面某个槽是否被指派的时候，需要遍历所有clusterNode的slots数组，时间为O(N)；用clusterState.slots就可以直接访问下标对应的clusterNode是否为空就可以，时间为O(1)。

使用clusterNode.slots记录是因为，传播该节点的处理槽信息，只需要将这个数组发送就可以，不用遍历clusterState.slots数组找出这个节点负责的槽然后再转发。

### addslot 命令的实现
遍历所有clusterState的slots数组，将对应的slots[i]指向自己；

再将当前节点自己的clusterNode的slots[i]的二进制位改为1.

## 集群中执行命令
client向节点发送数据库键相关命令时，会根据key计算属于哪个slot，然后判断当前节点是不是负责处理这个slot的。如果是，就直接执行命令；否则节点向客户端返回一个moved错误，client再根据moved错误的信息，将命令重定向到正确的节点。

### 计算key属于哪个slot
根据CRC16校验和，和按位与16383计算出一个在0-16383之间的树作为该key的slot index。

### 判断槽是否由当前节点处理
根据校验和计算出来的slot index，查找clusterState.slot[index]是否属于myself指针，如果是，则执行；如果不是则向客户端返回moved错误。

### moved错误
mvoed错误格式：MOVED SLOT IP:PORT

表示该SLOT是由IP:PORT节点负责的。客户端根据返回的IP/PORT，将命令重新转发到该节点去执行。

### 节点数据库的实现
节点数据库与单机数据库有区别：节点数据库只能使用0号数据库；单机数据库可以使用多个数据库。

clusterState还会保存slots-to-keys，一个zskiplist结构，用来保存key和slots之间的关系。这个跳跃表中，每个key的score都是一个槽号；所以可以通过这个跳跃表，对这些某些slot中的数据库key进行批量操作。（比如count该slot中有多少个key）。



## 重新分片
将某些key重新指派给其他节点负责的slot，就是重新分片。

重新分片通过redis提供的官方管理软件redis-trib负责执行；

1. 向目标节点发送 CLUSTER SETSLOT SLOT IMPORTING SOURCE-ID，让目标节点准备好导入槽slot的键值对；
2. 向源节点发送 CLUSTER SETSLOT SLOT MIRGRATING TARGET-ID，让源节点准备好将槽slot迁移到目标节点；
3. 向源节点发送 CLUSTER GETKEYSINSLOT SLOT COUNT 命令，获得最多count个槽slot的键值对的键名；
4. 对于每个获得的键名，都向源节点发送 MIRGRATE TARGET-ID TARGET-PORT KEY-NAME 0 TIMEOUT 将被选中的键从源节点迁移至目标节点。
5. 重复3和4，将所有属于slot的键值对都迁移到目标节点位置。
6. 向集群中任意节点发送 CLUSTER SETSLOT SLOT NODE TARGET-ID 命令，将slot指派给目标节点；其他节点会将该信息也发送给集群中的其他节点，这样集群就会知道SLOT已经指派给目标节点。



## ASK错误

重新分片过程中，源节点和目标节点在迁移一个slot的过程中，客户端刚好查询某个属于该slot的数据的时候，就可能会发送ASK错误。

此时：如果找不到key对应的数据，并且发现源节点正在迁移slot，表明key可能在目标节点（已经被迁移了），那么源节点就会向client发送一个ASK错误，指引客户端转向目标节点并且发送之前执行的命令。

通过clusterState中的importing_slots_from数组和migrating_slots_to数组，记录了当前节点正在从某个节点导入/迁移的slot，每个数组元素其指向一个clusterNode结构对象。

---

### ASK错误与MOVED错误
MOVED错误是永久性的，代表slot已经移交给另一个节点处理；以后每次遇到该slot的命令时，都会重定向到MOVED错误指向的节点处理；

ASK错误是临时性的，客户端只会在接下来一次的请求中将slot i的请求发送至ASK错误指示的节点；后面有关slot i的命令还是发送给目前处理slot i的节点，除非再次出现ASK错误。


## 复制与故障转移

redis集群中节点分为master和slave，master用于处理slot，slave复制master的结果。当master下线时，slave提升为master，继续处理slot。

### 设置从节点
命令：CLUSTER REPLICATE [NODE ID]：
让接受命令的节点成为NODE-ID的从节点。

1. 在clusterState.nodes字典中找到node-id对应的clusterNode结构，将自己的clusterState.myself.slaveof指向这个结构。

2. 修改clusterState.myself.flags，关闭REDIS_NODE_MASTER，打开REDIS_NODE_SLAVE，表示节点从主节点变成从节点。

3. 调用复制代码，开始对slaveof指向的clusterNode保存的IP:PORT进行复制。

### 故障检测
每个节点定时向集群中其他节点发送PING消息，当超过规定时间没有PONG消息返回，就标记为疑似下线（REDIS_NODE_PFAIL标识)。

集群中节点通过互相发送消息来交换集群中的各个节点的状态信息。

当主节点A通过消息得知主节点B认为主节点C已经进入了PFAIL状态时，主节点A会在自己的clusterState.nodes字典中找到主节点C对应的clusterNode结构，将主节点B的failure report添加到clusterNode结构的fail_reports链表里面。

如果一个集群里，半数以上处理slot的主节点认为C是PFAIL，那么主节点C会被标记为FAIL，然后将C标记为FAIL的节点会向集群中所有的节点广播一条C的FAIL消息，所以接收到该消息的节点会立即将C标记为下线。

### 故障转移
当一个从节点发现自己正在复制的主节点下线时，开始故障转移：

1. 复制下线主节点的所有从节点会有一个被选中执行 slaveof no one，称为新的主节点；
2. 新的主节点会撤销所有对已下线的主节点的slot的指派，将这些slot指派给自己。
3. 新的master向集群广播一条PONG消息，这条PONG消息会让集群中其他节点知道这个节点由从节点变成了主节点，并且这个主节点接管了原下线节点管理的slot。
4. 新主节点开始接受和自己负责处理的slot有关的命令请求，完成故障转移。

### 选举新的主节点

1. 每个集群维护一个config_epoch用于选举，是一个自增计数器，初始为0；
2. 当集群里的某个节点开始一次故障转移操作的时候，config_epoch自增1；
3. 每个config_epoch，集群中的master都只有一次投票的机会，第一个向该master请求投票的slave将获得该master的投票；
4. 当slave发现自己正在复制的master已下线，会向集群广播一条消息，要求所有收到消息且可以投票的master向这个slave进行投票；
5. 如果一个master具有投票权，并且未投票给其他节点，那么master会向要求投票的slave返回一套消息，表示支持该slave成为新的master；
6. 每个参与选举的slave都会通过接收master返回的消息，统计自己获得多少master的支持；
7. 如果slave获得N/2+1个master的支持，就会成为新的master。
8. 如果一个config_epoch里面没有slave获得足够多的投票，那么集群会进入一个新的config_epoch，再次进行选举，直到选举出结果为止。

## 发送消息 

* 消息头
    ![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200909105017.png)
    data指向消息正文。

    ![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200909105144.png)

    消息头中包括了发送者自身的一些消息，接收者可以根据消息头，在clusterNode中的nodes字典中找到对应的node进行更新。

### MEET/PING/PONG
MEET消息是客户端向服务器发送CLUSTER MEET命令时发送的；

PING消息是默认1秒从列表中选出5个节点中最长没有发送过PING的节点发送PING消息；

PONG消息是收到MEET/PING消息时为确认消息到达的回复。同时，节点也可以向集群广播自己的PONG消息，更新其他节点对自己的认识。

每次发送消息时，从已知节点列表中选出两个，保存到下面的结构数组里面。
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200909152511.png)

节点每次收到MEET/PING/PONG消息时，会根据clusterMsgDataGossip结构，判断自己是否首次接触该节点。

如果接受的节点发现传来的消息不在列表里，说明接受者是第一次接触到该节点，接受者会根据传来的消息进行握手。

如果被选中节点已经存在于列表，接受者就会根据消息记录的信息，更新对应的clusterNode结构。



### FAIL
但节点A认为节点B已经下线时，会向集群广播一条主节点B的FAIL消息，所有接收到该消息的节点都会将节点B标记为下线。

采取广播是因为Gossip协议需要一定延迟才能通知到集群所有节点，为了尽快进行故障转移，故采用广播。

FAIL消息正文只包含一个nodeName的属性，表示下线节点的名字。

### PUBLISH
client向服务端发送PUBLISH时，会向所有节点广播PUBLISH消息，所以所有节点都会执行  PUBLISH 命令。