---
title: MIT6.824-LAB2-Summary
date: 2020-11-04 22:05:09
categories: 
- 分布式
- 6.824
tags: [分布式,6.824]
---
6.824 lab的一些Q&A，以及面试遇到的一些问题。
<!---more--->

## 为什么RAFT日志不允许空洞

https://www.zhihu.com/question/36648084


日志的连续性蕴含了这样一条性质：如果两个不同节点上相同序号的日志，只要term相同，那么这两条日志必然相同，且这和这之前的日志必然也相同的。一条日志被提交，代表这条日志之前所有的日志都已被提交，一条日志可以被提交，代表之前所有的日志都可以被提交。这样方便leader进行日志的比对，同时保证leader中所有的日志都是commit了的日志。

## 描述一下Corner case，为什么有个“只能提交当前term的日志限制”

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20210324230612.png)

1. S1复制日志给部分节点，然后crash；
2. S5当选term3的leader；因为3,4都还没有log[2]，所以S5可以从S3，S4，S5自己获得3票，当选leader；
3. S5crash，S1重新当选term4的leader；如果这时候S1继续复制log[2]给S3，企图commit log[2]；如果S1复制给S3，OK，这时候log[2]应该是commit了的；但假如这时候S1挂掉了；S5重新当选leader，【因为这时候S2，S3，S4的log长度一致，但是更新的是S5的log[3]，所以是可以获得选票的】；
4. 这时候S5广播log[3]，就会对已经commit的log[2]进行覆盖，就会造成已经commit的log遭到回滚，就会出错；
5. 所以就需要做限制，只能提交当前term的日志。这时候我们尝试commit log[4]，如果发现可以commit，这时候我们更新commitIndex；间接地就已经commit log[2]；因为如果log[2]不能commit，我们不能直接commit log[4]，那我们可能需要同时将log[2]和log[4]进行发送，如果成功就更新commitIndex；这时候S5就选不了leader，因为它最新的logterm是3<4.


## 5个节点中有一个节点失效情况下，leader选举会出现什么问题？

可能会存在同时又两个candidate获得2张选票，然后达不到一半节点以上的票数，这时候需要重新选举；

通过随机选举超时可以尽可能避免这种情况的发生。

## 描述一下Prevote
prevote也可以避免某一个节点在产生网络分区的情况下，不停地发起选举，自增自己的term，然后强制其他节点重新选举的问题。


在PreVote算法中，Candidate首先要确认自己能赢得集群中大多数节点的投票，这样才会把自己的term增加，然后发起真正的投票。

PreVote算法解决了网络分区节点在重新加入时，会中断集群的问题。在PreVote算法中，网络分区节点由于无法获得大部分节点的许可，因此无法增加其Term。然后当它重新加入集群时，它仍然无法递增其Term，因为其他服务器将一直收到来自Leader节点的定期心跳信息。一旦该服务器从领导者接收到心跳，它将返回到Follower状态，Term和Leader一致。

