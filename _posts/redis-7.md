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

# 节点
redis会根据配置确定是否开启集群模式，集群模式下redis会继续沿用单机模式下的一些组件，比如使用rdb和aof持久化、使用serverCron函数进行周期操作等等。

通过cluster meet可以申请向某ip和port申请握手。

集群模式下有自己的独特的集群数据结构。

## 集群数据结构
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200830145107.png)

clusterNode结构如上，保存了包括名字、创建节点的时间、槽数组等属性。

clusterNode还保存了clusterLink结构，这个结构是节点的连接信息（像webserver中的httpconnection）

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200830145907.png)

同时，每个节点还会保存一个clusterState结构，记录当前节点的视角下，集群的整体状态。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200830150036.png)

该结构包括指向自己的指针，配置纪元（用于故障转移）、集群的状态（上线还是下线）、集群节点dict（key是名字，值为一个clusterNode结构的指针）等。

具体的结构内容示例：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200830150459.png)


## cluster meet
MEET/PING/PONG三次握手。

# 槽指派
## 槽指派信息记录
redis cluster通过分片的方式保存数据库中的键值对，整个数据库被分成16384个slot，每个key都属于其中之一。每个node可以处理1-16384个slot。

当数据库中有一个槽没节点处理时，集群处于下线状态；

通过cluster addslots命令，可以将一个或多个槽委派该节点负责。

clusterNode的numslots属性和slots数组记录了节点负责哪些槽。

slots数组是个二进制位数组，长度为16384/8个字节，按照对应i位上的二进制位值来判断节点是否负责处理槽i。

比如i位上是1，就认为负责处理slot1.

## 传播节点的槽指派信息
节点会将自己的slots数组和numslot传播给其他节点，告诉其他节点自己正在处理哪些slot。然后其他节点收到信息，会更新其对应的clusterNode的slots数组和numslot。

## 记录集群中所有槽的指派信息
clusterState中slots数组记录了所有槽的指派信息，数组指向一个clusterNode结构，表示第i位的slot由该clusterNode处理。

这种方式处理是为了防止查找上面某个槽是否被指派的时候，需要遍历所有clusterNode的slots数组，时间为O(N)；用clusterState.slots就可以直接访问下标对应的clusterNode是否为空就可以，时间为O(1)。

使用clusterNode.slots记录是因为，传播该节点的处理槽信息，只需要将这个数组发送就可以，不用遍历clusterState.slots数组找出这个节点负责的槽然后再转发。

## addslot 命令的实现
遍历所有clusterState的slots数组，将对应的slots[i]指向自己；

再将当前节点自己的clusterNode的slots[i]的二进制位改为1.

# 集群中执行命令
client向节点发送数据库键相关命令时，会根据key计算属于哪个slot，然后判断当前节点是不是负责处理这个slot的。如果是，就直接执行命令；否则节点向客户端返回一个moved错误，client再根据moved错误的信息，将命令重定向到正确的节点。

## 计算key属于哪个slot
根据CRC16校验和，和按位与16383计算出一个在0-16383之间的树作为该key的slot index。

## 判断槽是否由当前节点处理
根据校验和计算出来的slot index，查找clusterState.slot[index]是否属于myself指针，如果是，则执行；如果不是则向客户端返回moved错误。

## moved错误
mvoed错误格式：MOVED SLOT IP:PORT

表示该SLOT是由IP:PORT节点负责的。客户端根据返回的IP/PORT，将命令重新转发到该节点去执行。

## 节点数据库的实现
节点数据库与单机数据库有区别：节点数据库只能使用0号数据库；单机数据库可以使用多个数据库。

clusterState还会保存slots-to-keys，一个zskiplist结构，用来保存key和slots之间的关系。这个跳跃表中，每个key的score都是一个槽号；所以可以通过这个跳跃表，对这些某些slot中的数据库key进行批量操作。（比如count该slot中有多少个key）。



# 重新分片
将某些key重新指派给其他节点负责的slot，就是重新分片。

重新分片通过redis提供的官方管理软件redis-trib负责执行；

1. 向目标节点发送 CLUSTER SETSLOT SLOT IMPORTING SOURCE-ID，让目标节点准备好导入槽slot的键值对；
2. 向源节点发送 CLUSTER SETSLOT SLOT MIRGRATING TARGET-ID，让源节点准备好将槽slot迁移到目标节点；
3. 向源节点发送 CLUSTER GETKEYSINSLOT SLOT COUNT 命令，获得最多count个槽slot的键值对的键名；
4. 对于每个获得的键名，都向源节点发送 MIRGRATE TARGET-ID TARGET-PORT KEY-NAME 0 TIMEOUT 将被选中的键从源节点迁移至目标节点。
5. 重复3和4，将所有属于slot的键值对都迁移到目标节点位置。
6. 向集群中任意节点发送 CLUSTER SETSLOT SLOT NODE TARGET-ID 命令，将slot指派给目标节点；其他节点会将该信息也发送给集群中的其他节点，这样集群就会知道SLOT已经指派给目标节点。



# ASK错误

# 复制与故障转移

# 发送消息