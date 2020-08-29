---
title: redis设计与实现读书笔记(6)--多机数据库之sentinel高可用
date: 2020-08-22 17:40:53
tags: [redis]
keywords: [redis]
---
redis多级数据库主要有三部分：1、主从模式；2、sentinel高可用解决方案；3、redis集群。
这是第二部分，sentinel高可用解决方案。

<!---more--->



## sentinel模式

sentinel，redis高可用解决方案。由sentinel系统监测多个主服务器以及主服务器的从服务器，当某些主服务器不可用时，自动将主服务器下的某些从服务器切换为新的主服务器继续处理请求。

## sentinel的初始化

sentinel是redis一个特殊的服务器；没有redis服务器的数据库功能；有自己的专用代码（命令表），使用sentinel的时候会将命令表从redis命令更新到sentinel命令。

### sentinelState和sentinelRedisInstance结构
服务器状态保存在sentinelState结构中。
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200822150618.png)
其中masters记录了所有sentinel监视的主服务器实例，保存在字典数据结构中。这个字典的key是主服务器的名字，值是一个指向sentinelRedisInstance的结构的指针。

- - -

sentinelRedisInstance结构如下：包括runid，name，configepoch，sentinel地址、下线时间等等。
```C
typedef struct sentinelRedisInstance {
    int flags;      /* See SRI_... defines */
    char *name;     /* Master name from the point of view of this sentinel. */
    char *runid;    /* Run ID of this instance, or unique ID if is a Sentinel.*/
    uint64_t config_epoch;  /* Configuration epoch. */
    sentinelAddr *addr; /* Master host. */
    instanceLink *link; /* Link to the instance, may be shared for Sentinels. */
    mstime_t last_pub_time;   /* Last time we sent hello via Pub/Sub. */
    mstime_t last_hello_time; /* Only used if SRI_SENTINEL is set. Last time
                                 we received a hello from this Sentinel
                                 via Pub/Sub. */
    mstime_t last_master_down_reply_time; /* Time of last reply to
                                             SENTINEL is-master-down command. */
    mstime_t s_down_since_time; /* Subjectively down since time. */
    mstime_t o_down_since_time; /* Objectively down since time. */
    mstime_t down_after_period; /* Consider it down after that period. */
    mstime_t info_refresh;  /* Time at which we received INFO output from it. */
    dict *renamed_commands;     /* Commands renamed in this instance:
                                   Sentinel will use the alternative commands
                                   mapped on this table to send things like
                                   SLAVEOF, CONFING, INFO, ... */

    /* Role and the first time we observed it.
     * This is useful in order to delay replacing what the instance reports
     * with our own configuration. We need to always wait some time in order
     * to give a chance to the leader to report the new configuration before
     * we do silly things. */
    int role_reported;
    mstime_t role_reported_time;
    mstime_t slave_conf_change_time; /* Last time slave master addr changed. */

    /* Master specific. */
    dict *sentinels;    /* Other sentinels monitoring the same master. */
    dict *slaves;       /* Slaves for this master instance. */
    unsigned int quorum;/* Number of sentinels that need to agree on failure. */
    int parallel_syncs; /* How many slaves to reconfigure at same time. */
    char *auth_pass;    /* Password to use for AUTH against master & replica. */
    char *auth_user;    /* Username for ACLs AUTH against master & replica. */

    /* Slave specific. */
    mstime_t master_link_down_time; /* Slave replication link down time. */
    int slave_priority; /* Slave priority according to its INFO output. */
    mstime_t slave_reconf_sent_time; /* Time at which we sent SLAVE OF <new> */
    struct sentinelRedisInstance *master; /* Master instance if it's slave. */
    char *slave_master_host;    /* Master host as reported by INFO */
    int slave_master_port;      /* Master port as reported by INFO */
    int slave_master_link_status; /* Master link status as reported by INFO */
    unsigned long long slave_repl_offset; /* Slave replication offset. */
    /* Failover */
    char *leader;       /* If this is a master instance, this is the runid of
                           the Sentinel that should perform the failover. If
                           this is a Sentinel, this is the runid of the Sentinel
                           that this Sentinel voted as leader. */
    uint64_t leader_epoch; /* Epoch of the 'leader' field. */
    uint64_t failover_epoch; /* Epoch of the currently started failover. */
    int failover_state; /* See SENTINEL_FAILOVER_STATE_* defines. */
    mstime_t failover_state_change_time;
    mstime_t failover_start_time;   /* Last failover attempt start time. */
    mstime_t failover_timeout;      /* Max time to refresh failover state. */
    mstime_t failover_delay_logged; /* For what failover_start_time value we
                                       logged the failover delay. */
    struct sentinelRedisInstance *promoted_slave; /* Promoted slave instance. */
    /* Scripts executed to notify admin or reconfigure clients: when they
     * are set to NULL no script is executed. */
    char *notification_script;
    char *client_reconfig_script;
    sds info; /* cached INFO output */
} sentinelRedisInstance;

```
**较为重要的几个属性**

| 属性 | 含义 |
|:----------:|:-------------:|
flags|标识，记录了实例类型和实例状态
name|实例名称，默认为ip：port
runid|实例runid
config_epoch|故障转移的计数器
addr|IP地址和端口
down_after_period|判断为下线的时间条件
quorum|选举leader所需的投票数量
parallel_sync|故障转移时允许同时对新的主服务器进行同步的从服务器数量
timeout|刷新故障转移状态的最大时限
sentinels【dict类型】|监视这个master的sentinel服务器
slaves【dict类型】|master的slave

### 创建与主服务器的网络连接
主要创建两个连接，一个是向主服务器发送命令的，一个是用于订阅主服务器的hello频道的订阅连接。（原因是redis的订阅功能不会将消息保存在redis服务器中，所以要专门建一个连接处理这个频道发来的消息）

## 获取主服务器消息
sentinel默认以10s一次的频率通过命令连接向主服务器发送INFO命令，来获取主服务器的当前信息。

通过INFO命令可以获取到的信息包括：runid、role、主服务器下的所有从服务器的信息。主服务器下的所有从服务器消息保存在名称为slaves的dict数据结构中，每个dict的值和上述一样，是一个指向sentinelRedisInstance结构的指针。

## 获取从服务器消息

sentinel会对从服务器同样建立命令连接和订阅连接，同样会发送INFO命令获取从服务器信息。

## 向主服务器和从服务器发送信息

默认2s一次的频率向主服务器和从服务器通过命令连接发送PUBLISH命令：

PUBLISH __ sentinel __::hello </args/>

这个命令向__ sentinel __::hello频道发送了一条信息。命令参数及含义如下：

| 参数 | 含义 |
|:----------:|:-------------:|
s_ip|sentinel ip
s_port|sentinel port
s_runid|sentinel runid
s_epoch|sentinel config epoch
m_name|master name
m_ip|master ip
m_port|master port
m_epoch|master current epoch

如果向从服务器发送消息，就会返回从服务器正在复制的主服务器信息。


## 接收来自主服务器和从服务器的频道消息

sentinel与主从服务器建立订阅连接后，会向服务武器发送SUBSCRIBE __ sentinel __:hello，订阅该频道。后续可以通过这个频道接收信息。

监视同一个服务器的多个sentinel服务器都会接收到其他sentinel服务器通过发送来的信息。

也就是说：服务器通过命令连接发送信息到频道   --->    服务器通过订阅连接接收信息

## 更新sentinels字典
党一个sentinel接收到其他sentinel发来的信息时，目标sentinel会从信息系中分析出
1. 与sentinel有关的参数；
2. 与master有关的参数

根据取出的master参数，在自己的sentinelState的master字典中找到对应的sentinelRedisInstance类型master实例，更新或创建该master实例的sentinel实例。

当发现新的sentinel，创建完毕实例后，会建立命令连接，__不创建订阅连接__。因为sentinel主要通过接受master或slave传来的信息发现sentinel，随后直接与sentinel建立命令连接就可以通信。

## 检测下线状态

### 主观下线检测
1s一次向所有创建了连接的实例发送PING命令，如果在指定时间内收不到有效回复（PONG/LOADING/MASTERDOWN)，就会修改这个实例对应的flag，标识实例已经主观下线。

### 检查客观下线
为了确认服务器是否真的下线，会向其他sentinel进行询问。
1. 发送SENTINEL is-master-down-by-addr [ip] [port] [current-epoch] [runid]命令；

    | 参数 | 含义 |
    |:----------:|:-------------:|
    ip|sentinel认为下线的master的ip
    port|主管下线master port
    current_epoch|sentinel当前的config epoch，用于选举leader
    runid|*或真正runid，*表示检测master是否主观下线，runid则用于选举leader


2. 接收上述命令 ，取出命令的参数，检查master是否下线，然后返回三个参数[down state] [leader runid] [leader epoch]

    | 参数 | 含义 |
    |:----------:|:-------------:|
    down_state|检查结果，0表示未下线，1表示下线
    leader_runid|同上述runid
    leader_epoch|sentinel当前的config epoch，用于选举leader

3. 接收命令回复：根据命令的内容，统计其他sentinel同意master下线的数量，当超过配置数目的时候，就修改flags，认定master客观下线。但不同的sentinel判断客观下线的判断条件可能不同。对一个sentinel认为客观下线，对另一sentinel却未必。

## 选举leader sentinel

当一个master被认定为客观下线时，sentinel会进行协商选出leader对下线的master进行故障转移。

1. 每次进行选举，config_epoch都会自增1.
2. 每个发现master进入客观下线的sentinel都会要求其他sentinel将其设置为局部leader sentinel；在一个epoch里，sentinel只有一次可以将某另一个sentinel设置为局部leader。在这个epoch里，局部leader设置后不能更改。
3. 源sentinel向另一个sentinel（目标sentinel）发送SENTINEL is-master-down-by-addr 命令。当runid不为 * ，就表示源sentinel请求目标sentinel将自身设置为局部leader。
4. 目标sentinel接收到请求后，只接受最先到来的请求，将一个sentinel设置为局部leader，其他后续发送来的请求都会拒绝。然后发送回复，回复的leader_runid将包含目标sentinel认为的leader sentinel的runid，leader_epoch将包含其config epoch。
5. 源sentinel接收到回复，会检查leader_runid是否和自身runid一致，如果是，就表明目标sentinel将自身设置为局部leader。
6. 如果有某个sentinel被半数以上的sentinel设置为局部leader，该sentinel就会正式成为leader sentinel。
7. 如果在给定时间内，未选出leader sentinel，就会在一段时间后再次选举，知道选出leader为止。


## 故障转移

1. 从下线的主服务器的从服务器里挑选一个转换为主服务器；
2. 让已下线的主服务器的所有从服务器复制新主服务器；
3. 

