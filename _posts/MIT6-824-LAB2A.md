---
title: MIT6.824-LAB2A
date: 2020-10-25 22:07:45
categories: 
- 分布式
- 6.824
tags: [分布式,6.824]
---

一个多月没更blog，就是因为开了这个坑。。

<!---more--->

## 概要
6.824是一门神课，在中外都是顶顶大名的了。可惜当年还年轻，还不知道这门课有多难得，对OS和网络也不太熟悉，就从基础开始补课，再刷了一个C++的web server，打了一阵子的leetcode终于开始了。

6.824顶顶大名的lab就是手写RAFT支持的K/V数据库。在网上看似有很多资料，但是很多资料和代码都不一定是完美的。

做lab之前，一定要看课程的视频，论文也要认真研读。做的时候不能错过任何一个细节。

附上课程首页：http://nil.csail.mit.edu/6.824/2020/index.html

课程lab源码git仓库：
```Shell
git clone git://g.csail.mit.edu/6.824-golabs-2020 6.824
```
lab2 instruction: http://nil.csail.mit.edu/6.824/2020/labs/lab-raft.html


## RAFT算法
论文中将RAFT分为三部分：leader选举、日志复制、安全限制。

整个RAFT的集群节点就是分为三个角色，leader、candidate、follower，数据的复制只会从leader流向follower，follower在选举超时时，会变为candidate进行选举，选举成功的节点成为leader。所以leader的选举至关重要。

![raft状态机](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20201101164648.png)


## lab2A
LAB2A要实现的是第一个部分，也就是leader选举。

整个lab的流程其实就是通过Make()函数，创建几个节点，然后节点之间通过一定的时间事件驱动，向其他节点发送RPC消息，然后我们再在RPC处理函数中处理接收到的RPC消息。在LAB2中两个RPC分别是选举RPC（voteRPC）和日志复制RPC（AppendEntriesRPC）。

根据论文的Figure2和5.2的规则，我们可以很容易写出voteRPC的流程：

1. follower超过随机的选举时间，转变为candidate；
2. candidate进行选举，将currentTerm += 1，投票给自己；
3. candidate发送RPC给其他节点；
4. 其他节点如果收到RPC，首先要看RPC是不是符合要求；如果收到RPC的term比自己的term小，reply false；
5. 如果收到term更大，自身变为follower，但注意此时不要重置选举时间；
6. 检查是否满足投票条件（5.4节，日志是最新的）；
7. 如果是，投票，reply true，重置选举时间。
8. 发出RPC的节点检查reply true的数量，如果超过节点一半，就转变为leader。

接收方RPC函数：
```go
func (rf *Raft) RequestVote(args *RequestVoteArgs, reply *RequestVoteReply) {
	// Your code here (2A, 2B).

	rf.lock()
	defer rf.unlock()

	//Log().Debug.Printf("server %d recv voteRequest from candidate server: %d args.term:%d, rf.term:%d vote for:%d, candidate:%d",
	//	rf.me,args.CandidateId,args.Term,rf.currentTerm,rf.voteFor,args.CandidateId)

	//If RPC request or response contains term T > currentTerm:
	//set currentTerm = T, convert to follower (§5.1)
	//For example, if you have already voted in the current term,
	//and an incoming RequestVote RPC has a higher term that you,
	//you should first step down and adopt their term (thereby resetting votedFor),
	//and then handle the RPC, which will result in you granting the vote!
	if args.Term > rf.currentTerm{
		//Log().Info.Printf("s%d, reqvote args.Term > rf.currentTerm",rf.me)
		rf.changeToFollower(args.Term)
	}

	reply.Term = rf.currentTerm
	reply.VoteGranted = false


	//Reply false if term < currentTerm (§5.1)
	if args.Term < rf.currentTerm || ( rf.voteFor != -1 && rf.voteFor != args.CandidateId){
		//Log().Warning.Printf("reqvote args.Term < rf.currentTerm")
		return
	}


	//shorter logs or logs aren't up to date
	//restrict vote according to figure 5.4
	// If votedFor is null or candidateId, and candidate’s log is at
	// least as up-to-date as receiver’s log, grant vote (§5.2, §5.4)
	if args.LastLogTerm < rf.getLastLogTerm() || (args.LastLogTerm == rf.getLastLogTerm() && args.LastLogIndex < rf.getLastLogIndex()){
		//Log().Warning.Printf("restrict log is up to date")
		return
	}

	rf.voteFor = args.CandidateId
	reply.VoteGranted = true
	rf.role = Follower
	rf.persist()
	//you should only restart your election timer
	//if a) you get an AppendEntries RPC from the current leader (i.e., if the term in the AppendEntries arguments is outdated,
	//you should not reset your timer);
	//b) you are starting an election; or
	//c) you grant a vote to another peer.
	rf.sendCh(rf.voteCh)
	//Log().Debug.Printf("server %d vote for server %d",rf.me,args.CandidateId)

}
```

发送方RPC函数：
```go
func (rf *Raft) startElection() {
	//Log().Info.Printf("server %d start election..",rf.me)
	//defer rf.persist()

	rf.lock()
	args := RequestVoteArgs{
		Term:         rf.currentTerm,
		CandidateId:  rf.me,
		LastLogIndex: rf.getLastLogIndex(),
		LastLogTerm:  rf.getLastLogTerm(),
	}
	rf.unlock()

	
	for i:= range rf.peers{
		if i == rf.me{
			continue
		}

		go func(peerIdx int, args *RequestVoteArgs) {
			var reply RequestVoteReply
			ok := rf.sendRequestVote(peerIdx,args,&reply)
			if ok{
				rf.lock()
				defer rf.unlock()

				//From experience, we have found that by far the simplest thing to do is to first record the term in the reply
				//(it may be higher than your current term), and then to compare the current term with the term you sent in your original RPC.
				//If the two are different, drop the reply and return.
				//Only if the two terms are the same should you continue processing the reply.
				if rf.currentTerm != args.Term || rf.role != Candidate{
					return
				}

				//If RPC request or response contains term T > currentTerm:
				//set currentTerm = T, convert to follower (§5.1)
				if reply.Term > rf.currentTerm{
					rf.changeToFollower(reply.Term)
					return
				}

				//If votes received from majority of servers: become leader
				if reply.VoteGranted {
					rf.votesGranted += 1
					if rf.votesGranted > len(rf.peers)/2{
						rf.changeToLeader()
						rf.sendCh(rf.voteCh)
					}
				}
			}
		}(i,&args)
	}
}
```