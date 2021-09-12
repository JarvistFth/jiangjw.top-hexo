---
title: MIT6-824-LAB4
date: 2020-12-11 21:41:44
categories: 
- 分布式
- 6.824
tags: [分布式,6.824]
---

6.824 lab4
<!---more--->

## 整体结构

raft client -> metaData query service -> kv sharded group server

metaData query service: 
1. 负责描述每个key对应的value属于哪个group；
2. 响应每次的JOIN/LEAVE/MOVE/QUERY 请求，对应group的变化

kv sharded group server:
1. 分片存储所有数据；
2. 响应client请求；
3. 周期性请求meta service；如果meta有变化，就进行对应的data migration， 更新自己的config；

## rebalance
给每个分片分配平均的group数目，减少它们的差异。

方法：每次shard数目减少以后，将该shard的group加到最少的group；每次shard数目增加，从group数目最多的shard中取一个分出去个新的shard。

## 每个sharded group server

1. queryMeta， 去拉最新的config；
2. 拒绝client传来的不属于自己shard的代码；
3. 某个shard发现meta发生了变更，就主动去对应的另一个shard去fetch group 对应的data；
4. 