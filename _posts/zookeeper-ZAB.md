---
title: zookeeper-ZAB
date: 2020-11-07 23:18:05
categories: 
- 分布式
- zookeeper
tags: [分布式,zookeeper,ZAB]
---

zab是专门为zookeeper实现的一个一致性算法。
<!---more--->

## zookeeper 和 raft

raft是一个lib，一个算法用于帮助构建分布式系统；
zk是一个帮助别人协调分布式系统的应用，使用zk提供的api就可以。

## why zk？
考虑raft，我们在raft集群中加server是不是会获得更高的性能？

答案是并不会，因为server变多，leader要复制的log就越多，leader反而成为了瓶颈，会变慢；

让读请求直接让follower处理？该follower可能会和其他server处于另外的网络分区（split），无法获得最新副本，不能满足linearizable。

所以 zk 让读请求在zk节点上本地执行，虽然zk的读请求不满足线性一致性；但是有其他的一些保证。

## zk guarantee 

1. 写请求线性一致性，读请求不满足线性一致性；
2. 单一 client 按照FIFO order处理请求。

## 怎么实现guarantee？

### linearizable write
follower收到写请求，转发给leader，leader通过zab协议，保证将写请求proposal - commit 提交到各个follower中。当follower commit了以后，才回复client。

### FIFO  order

单一client维护一个队列，读和写请求都只会在zk server回复后才继续执行；写请求的回复上面说了；回复后再进行读取也能保证读到最新的state。

同时，client在发出读请求还会记录上次读到的最新的zxid；如果读的follower的zxid不是最新的，同样会阻塞。

## zk提供功能：

1. test and set

2. configuration

3. master election

---

## zk数据模型

unix like znode，分为 永久节点、临时节点、顺序节点。

永久节点只有调用delete才会删除；
临时节点在client与server的session失效、client断开连接，就会消失；
顺序节点会在创建的时候在末尾增加一系列递增的数字。

---

## zk如何保证Atomic 操作

zk可能会读到过期数据，所以两个client同时的get 和 put 并不是原子的。

解决：

```Python
while True:
    x,v = get("f")
    x = x + 1
    if set("f",x+1,v):
        break
```

test & set的一个过程，只有获取到set成功以后才会break出循环，自旋锁。

这种transaction，zk里面成为 mini transaction，只操作一个数据，保证了atomic。

---

## 问题：herd effect

上述会产生惊群效应，每次都会唤醒所有client争抢这个set函数的操作；所以复杂度是O(n^2)。

同样的还有这种LOCK：也会产生惊群效应，让所有client都进行争抢这个锁。

```Python

while True:
    if create("f",ephem = True)
        return # get lock here, return 

    if exist("f",watch = True)
        wait() # some one holds the lock
```

---

## 解决herd effect

利用SEQ-node：

```Python

lock = create("f-lock",ephem = True, SEQ = True)
while True:
    nodes = listing("f-lock*")
    if lock is lowest node in nodes:
        return #get the lock
    
    if exist(nodes[0].num < lock.num, watch = True):
        # zk has next lower file num in listing, wait
        # when watch event comes, re-listing file in zk and check the lowest file num
        wait()

```



