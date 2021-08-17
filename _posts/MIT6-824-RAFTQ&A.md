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

## paxos和raft区别

1. basic paxos， 多个proposer，3个阶段，prepare、promise、accept；可能会存在多个proposer轮流发送prepare造成死锁；
2. multi-paxos，basic-paxos的优化，在有一个proposer连续发送accept后，其他节点会拒绝prepare请求；相当于选出了一个leader负责去发送accept；可以跳过prepare和promise阶段；
3. raft的leader要求日志是连续的，paxos则没有要求，所有人都可以成为proposer；所以paxos在选出一个proposer以后，需要去做日志的同步；重新执行prepare阶段补全日志；
4. Raft保证日志的连续性使得leader向follower同步日志时可以进行快速的比对；因为这条日志已经commit的话，前面的所有日志都是已经commit了的，是连续的。


## readIndex机制
缓解read操作也要走leader的appendEntry的情况；

1. client往leader发送读请求的时候，检查一下自己是不是还是真的leader；
2. 然后记录当前commitIndex，收到大多数成功回复后，等applyIndex > commitIndex；
3. 返回结果。

follower的readIndex：向leader要readIndex的值，leader接到请求后，向其他节点发送no-op entry，检查自己的地位是否合法以及获取commitIndex；然后将commitIndex作为readIndex回复；follower在log到了readIndex后，回复客户端。

## raft的leader失效判断和lease机制

leader每次发送心跳的时候，都会记录一个时间点lease-start；只要大部分follower节点收到这次心跳，那么可以认为这次leader有效期为lease-start+election_timeout。（在这期间，其他follower保证不会参与leader选举，防止脑裂双主）。

问题：缺乏全局时钟，不同node之间的时钟可能有误差；如果误差过大，就会造成lease期限不一致，就可能出问题。