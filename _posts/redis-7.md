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

# 集群中执行命令

# 重新分片

# ASK错误

# 复制与故障转移

# 发送消息