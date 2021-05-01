---
title: MIT6-824-LAB3B
date: 2020-12-11 21:41:44
categories: 
- 分布式
- 6.824
tags: [分布式,6.824]
---

6.824 lab3B 思路和知识点整理。这里很多小坑小洼，其中那些snapshot以后要将下标对准就有很多坑，时刻要注意会不会数组越界等等。
<!---more--->

## 整体描述
lab3A做完以后，就是一个基于内存的raft-kv数据库；

现在要做的，就是将内存里面的东西定期归档，相当于redis里面的RDB save。恢复的时候直接从归档文件恢复。

## 代码分析

kv server需要在接近规定的max raft state的时候主动snapshot；

lab给出的persister提供了SaveStateAndSnapshot()和ReadSnapshot()方法，所以在对应的时候调用就可以了。

在主动做snapshot的时候，将要持久化的东西都持久化，同时还要让raft层面的log开始做snapshot，舍弃掉直到lastIncludeIndex的日志。


## 一些注意点：

1. 下标的更新；之前做lab2的时候，由于没有snapshot，所以所有的raft log的下标都是从最开始开始的；在做了snapshot以后，log数组清空，但是log的下标是从lastIncludeIndex开始的，这个Index也会持久化到raft state中。所以我们在访问log的term或者log本身的时候，需要做一个相对于 lastIncludeIndex的下标处理。

```go

// get some log index or terms
func (rf *Raft) getLastLogIndex() int{
	return rf.getLogsLength() - 1
}

func (rf *Raft) getLastLogTerm() int {
	back := rf.getLastLogIndex()
	return rf.getTerm(back)
}

func (rf *Raft) getPrevLogIndex(peeridx int) int {
	return rf.nextIndex[peeridx] - 1
}

func (rf *Raft) getPrevLogTerm(peeridx int) int {
	previdx := rf.getPrevLogIndex(peeridx)
	return rf.getTerm(previdx)
}

func (rf *Raft) getTerm(idx int) int {
	if idx < rf.lastIncludeIndex{
		return -1
	}
	return rf.getLog(idx).Term
}

func (rf *Raft) getLog(idx int) LogEntry {
	return rf.logs[idx - rf.lastIncludeIndex]
}

func (rf *Raft) getLogsLength() int {
	return len(rf.logs) + rf.lastIncludeIndex
}
```

2. takeSnapshot的时候，参照redis RDB的方法，另起一个goroutine去做takeSnapshot；如果不起goroutine可能会造成死锁（我在测的时候没有做多次测试，有的时候可能不会碰到？）。所以参考redis的方法，最好还是另起一个子进程或者线程去做snapshot。

3. 论文里面提到leader有个发起installSnapshot的RPC；需要有这个RPC是因为：假如这时候leader往一个follower去发一个index 为 x 的log，但是，follower还没回复leader OK，所以这时候leader会一致发送index 为x的log；如果这时候刚好做了snapshot，index为x的log已经被include进去了，这时候就不应该发送log，而是应该发送snapshot。

```go
//Log().Debug.Printf("nextIndex[%d]:%d, lastIncludeIdx:%d",peerIdx,rf.nextIndex[peerIdx],rf.lastIncludeIndex)
	//如果要发送给follower的日志已经被压缩了，就直接发送快照
	if rf.nextIndex[peerIdx] - rf.lastIncludeIndex < 1{
		//Log().Debug.Printf("send install snapshot to %d",peerIdx)
		rf.unlock()
		rf.InstallSnapshot(peerIdx)
		return
    }
```

4. 更新commitIndex和lastApplied的时候，有可能会出现不满足线性一致性或者数组越界的情况。打log可以看到lastApplied有时候会小于lastIncludeIndex，意思是我们取lastApplied的Index的log已经被snapshot进去了。所以我们取commitIndex和lastApplied的时候就要注意，取它们和lastIncludeIndex的较大值。
5. 同样的，在handleInstallSnapshotRPC的时候，也要同样的更新，并且如果发过来的lastIncludeIndex是小于lastApplied的话，就直接返回，不要让这个log携带的command再次应用到状态机上。

```go
func (rf *Raft) updateLastApplied() {

	rf.lastApplied = Max(rf.lastIncludeIndex,rf.lastApplied)
	rf.commitIndex = Max(rf.lastIncludeIndex,rf.commitIndex)

	for rf.lastApplied < rf.commitIndex {
		rf.lastApplied++
		//Log().Debug.Printf("lastApplied:%d, lastIncludeIdx:%d",rf.lastApplied,rf.lastIncludeIndex)
		curLog := rf.getLog(rf.lastApplied)
		applyMsg := ApplyMsg{
			true,
			curLog.Cmd,
			rf.lastApplied,
			nil,
		}
		rf.applyCh <- applyMsg
	}

}
```
handleInstallSnapshotRPC()：
```go 
    rf.persister.SaveStateAndSnapshot(rf.encodeRaftState(),args.Data)

	rf.lastApplied = Max(rf.lastApplied,rf.lastIncludeIndex)
	rf.commitIndex = Max(rf.commitIndex,rf.lastIncludeIndex)

	if rf.lastApplied > rf.lastIncludeIndex{
		return
	}
	//reset state machine using snapshot content
	applyMsg := ApplyMsg{
		CommandValid: false,
		SnapShot:     args.Data,
	}
	rf.applyCh <- applyMsg
    
```


## 修改后的lab2C， raft appendEntry代码

```go
package raft

import (
	"fmt"
	"sort"
)

type AppendEntriesArgs struct {
	Term int
	LeaderId int

	PrevLogIndex int
	PrevLogTerm int

	Entries []LogEntry

	LeaderCommitIdx int
}

func (args AppendEntriesArgs) String() string {
	return fmt.Sprintf("term:%d, leaderid:%d, prevIndex:%d, prevTerm:%d, leaderCommitIdx:%d",
		args.Term,args.LeaderId,args.PrevLogIndex,args.PrevLogTerm,args.LeaderCommitIdx)
}

type AppendEntriesReply struct {
	Term int
	Success bool

	ConflictIndex int
	ConflictTerm int
}

func (reply AppendEntriesReply) String() string {
	if reply.Success{
		return fmt.Sprintf("reply.term %d, success:%s",reply.Term,"true")
	}
	return fmt.Sprintf("reply.term %d, reply.success:%s",reply.Term,"false")
}

func (rf *Raft) RequestAppendEntries(args *AppendEntriesArgs, reply *AppendEntriesReply) {
	// Your code here (2A, 2B).

	rf.lock()
	defer rf.unlock()
	defer rf.sendCh(rf.heartbeatCh)
	defer rf.persist()

	//Log().Debug.Printf("server %d recv append entries from leader server %d",rf.me,args.LeaderId)



	if args.Term > rf.currentTerm{
		//Log().Info.Printf("args.Term %d > rf.currentTerm %d",args.Term,rf.currentTerm)
		rf.changeToFollower(args.Term)
	}

	reply.Term = rf.currentTerm
	reply.Success = false
	reply.ConflictIndex = -1
	reply.ConflictTerm = -1

	if args.Term < rf.currentTerm{
		//Log().Warning.Printf("args.Term %d < rf.currentTerm %d",args.Term,rf.currentTerm)
		return
	}


	//Reply false if log doesn’t contain an entry at prevLogIndex
	//whose term matches prevLogTerm (§5.3)

	//shorter log, conflict idx = len(rf.logs)

	//如果一个follower在日志中没有prevLogIndex，它应该返回conflictIndex=len(log)和conflictTerm = None。
	prevLogTerm := -1
	if args.PrevLogIndex >= rf.lastIncludeIndex && args.PrevLogIndex <= rf.getLastLogIndex() {
		prevLogTerm = rf.getTerm(args.PrevLogIndex)
	}

	logLength := rf.getLogsLength()


	if prevLogTerm != args.PrevLogTerm{
		reply.ConflictIndex = logLength
		//如果一个follower在日志中没有prevLogIndex，它应该返回conflictIndex=len(log)和conflictTerm = None。
		if prevLogTerm == -1{
			return
		}else{
			//如果一个follower在日志中有prevLogIndex，但是term不匹配，
			//它应该返回conflictTerm = log[prevLogIndex].Term，
			//并且查询它的日志以找到第一个term等于conflictTerm的条目的index。
			reply.ConflictTerm = prevLogTerm
			for i:= rf.lastIncludeIndex; i<logLength; i++{
				if rf.getTerm(i) == prevLogTerm{
					reply.ConflictIndex = i
					break
				}
			}
			return
		}
	}

	//now has the same previdx and logs[previdx].term, but its term may be different
	//Append any new entries not already in the log
	idx := args.PrevLogIndex
	for i,e := range args.Entries{
		idx++
		if idx < logLength{
			if rf.getLog(idx).Term == e.Term{
				continue
			}else{
				//If an existing entry conflicts with a new one (same index
				//but different terms), delete the existing entry and all that
				//follow it (§5.3) Figure 7 (f)
				rf.logs = rf.logs[:idx - rf.lastIncludeIndex]
			}
		}
		rf.logs = append(rf.logs,args.Entries[i:]...)
		break
	}

	//If leaderCommit > commitIndex, set commitIndex =
	//min(leaderCommit, index of last new entry)

	if rf.commitIndex < args.LeaderCommitIdx{
		//Log().Debug.Printf("follower commit, rf.commit:%d, arg.leadercommit:%d",rf.commitIndex,args.LeaderCommitIdx)
		rf.commitIndex = Min(args.LeaderCommitIdx,rf.getLastLogIndex())
		//rf.startApply <- 1
		rf.updateLastApplied()
	}
	reply.Success = true
}

func (rf *Raft) sendAppendEntries(server int, args *AppendEntriesArgs, reply *AppendEntriesReply) bool {
	//Log().Info.Printf("server %d sends append entries to server %d", rf.me,server)
	ok := rf.peers[server].Call("Raft.RequestAppendEntries", args, reply)
	return ok
}

//lock before using
func (rf *Raft) broadCast() {
	//Log().Info.Printf("server %d start broadcast heartbeats, rf.nextidx:%d, rf.matchidx:%d",rf.me,rf.nextIndex,rf.matchIndex)

	for i:=range rf.peers {
		if i == rf.me {
			continue
		}
		go rf.sendAppendEntriesRPC(i)
	}
}

func (rf *Raft) sendAppendEntriesRPC(peerIdx int) {
	reply := AppendEntriesReply{}

	rf.lock()

	if rf.role != Leader {
		rf.unlock()
		return
	}
	//Log().Debug.Printf("nextIndex[%d]:%d, lastIncludeIdx:%d",peerIdx,rf.nextIndex[peerIdx],rf.lastIncludeIndex)
	//如果要发送给follower的日志已经被压缩了，就直接发送快照
	if rf.nextIndex[peerIdx] - rf.lastIncludeIndex < 1{
		//Log().Debug.Printf("send install snapshot to %d",peerIdx)
		rf.unlock()
		rf.InstallSnapshot(peerIdx)
		return
	}

	prevLogIndex := rf.getPrevLogIndex(peerIdx)
	//Log().Debug.Printf("append log prevLogIdx:%d, lastIncludeIdx:%d",prevLogIndex, rf.lastIncludeIndex)
	args := AppendEntriesArgs{
		Term:            rf.currentTerm,
		LeaderId:        rf.me,
		PrevLogIndex:    prevLogIndex,
		PrevLogTerm:     rf.getPrevLogTerm(peerIdx),
		Entries:         append(make([]LogEntry, 0), rf.logs[rf.nextIndex[peerIdx]-rf.lastIncludeIndex:]...),
		LeaderCommitIdx: rf.commitIndex,
	}
	rf.unlock()
	//Log().Debug.Printf("prevLogIdx:%d, argLogLen:%d",prevLogIndex,len(args.Entries))
	//Log().Debug.Printf("send hb to %d",peerIdx)

	ok := rf.sendAppendEntries(peerIdx, &args, &reply)
	if ok {

		rf.lock()
		defer rf.unlock()

		//From experience, we have found that by far the simplest thing to do is to first record the term in the reply
		//(it may be higher than your current term), and then to compare the current term with the term you sent in your original RPC.
		//If the two are different, drop the reply and return.
		//Only if the two terms are the same should you continue processing the reply.

		if rf.role != Leader || rf.currentTerm != args.Term {
			return
		}

		//If RPC request or response contains term T > currentTerm:
		//set currentTerm = T, convert to follower (§5.1)
		if reply.Term > args.Term {
			rf.changeToFollower(reply.Term)
			return
		}

		if reply.Success {
			//If successful: update nextIndex and matchIndex for
			//follower (§5.3)
			rf.matchIndex[peerIdx] = args.PrevLogIndex + len(args.Entries)
			rf.nextIndex[peerIdx] = rf.matchIndex[peerIdx] + 1
			rf.updateCommitIdx()
			return
		} else {

			//If AppendEntries fails because of log inconsistency:
			//decrement nextIndex and retry (§5.3)

			//一旦收到一个冲突的响应，leader应该首先查询其日志以找到conflictTerm。
			//如果它发现日志中的一个具有该term的条目，它应该设置nextIndex为日志中该term内的最后一个条目的索引后面的一个。
			foundConflictTerm := false
			if reply.ConflictTerm != -1 {
				for i := args.PrevLogIndex; i >= 1; i-- {
					if rf.getTerm(i) == reply.ConflictTerm {
						rf.nextIndex[peerIdx] = i + 1
						foundConflictTerm = true
						break
					}
				}
			}
			//如果它没有找到该term内的一个条目，它应该设置nextIndex=conflictIndex。

			if !foundConflictTerm {
				//leader may take snapshot, and follower reply.conflict index may less than logs length
				rf.nextIndex[peerIdx] = Min(reply.ConflictIndex,rf.getLogsLength())
			}
		}
	}
}

//lock before calling
func (rf *Raft) updateCommitIdx() {
	rf.matchIndex[rf.me] = rf.getLastLogIndex()
	//If there exists an N such that N > commitIndex, a majority
	//of matchIndex[i] ≥ N, and log[N].term == currentTerm:
	//set commitIndex = N (§5.3, §5.4).
	tmp := make([]int, len(rf.matchIndex))
	copy(tmp,rf.matchIndex)
	sort.Ints(tmp)
	n := (len(rf.peers) - 1) / 2
	N := tmp[n]

	if N > rf.commitIndex && rf.getTerm(N) == rf.currentTerm{
		rf.commitIndex = N
		//Log().Debug.Printf("server %d commit index update to %d",rf.me,tmp[n])
		//rf.startApply <- 1
		rf.updateLastApplied()

	}
}

//lock before calling
func (rf *Raft) updateLastApplied() {

	rf.lastApplied = Max(rf.lastIncludeIndex,rf.lastApplied)
	rf.commitIndex = Max(rf.lastIncludeIndex,rf.commitIndex)

	for rf.lastApplied < rf.commitIndex {
		rf.lastApplied++
		//Log().Debug.Printf("lastApplied:%d, lastIncludeIdx:%d",rf.lastApplied,rf.lastIncludeIndex)
		curLog := rf.getLog(rf.lastApplied)
		applyMsg := ApplyMsg{
			true,
			curLog.Cmd,
			rf.lastApplied,
			nil,
		}
		rf.applyCh <- applyMsg
	}

}


```

## lab2加入snapshotRPC代码

```go
package raft

type InstallSnapshotArgs struct {
	Term int
	LeaderId int

	LastIncludeIndex int //snapshot replace all and include this index logs
	LastIncludeTerm int //term of lastIncludeIndex's log

	Data []byte
}

type InstallSnapshotReply struct {
	Term int
}

func (rf *Raft) TakeSnapshot(index int, snapshot []byte)  {
	rf.lock()
	defer rf.unlock()
	if index <= rf.lastIncludeIndex{
		return
	}
	//Log().Debug.Printf("SaveStateAndSnapshot, lastIncludeIndex:%d",rf.lastIncludeIndex)

	rf.logs = append(make([]LogEntry,0),rf.logs[index - rf.lastIncludeIndex:]...)
	rf.lastIncludeIndex = index
	rf.lastIncludeTerm = rf.getTerm(index)
	rf.persister.SaveStateAndSnapshot(rf.encodeRaftState(),snapshot)
}

func (rf *Raft) SendInstallSnapshotRPC(server int, args *InstallSnapshotArgs, reply *InstallSnapshotReply) bool {
	ok := rf.peers[server].Call("Raft.RequestInstallSnapshot",args,reply)
	return ok
}

func (rf *Raft) RequestInstallSnapshot(args *InstallSnapshotArgs, reply *InstallSnapshotReply) {
	rf.lock()
	defer rf.unlock()
	//Log().Debug.Printf("follower %d snapshot",rf.me)
	reply.Term = rf.currentTerm

	if args.Term < rf.currentTerm{
		return
	}

	if args.Term > rf.currentTerm{
		rf.changeToFollower(args.Term)
	}
	rf.sendCh(rf.heartbeatCh)

	if args.LastIncludeIndex <= rf.lastIncludeIndex{
		return
	}
	Log().Debug.Printf("follower snapshot")

	if args.LastIncludeIndex >= rf.getLogsLength() - 1{
		//discard all the logs
		rf.logs = []LogEntry{{
			Term: args.LastIncludeTerm,
			Cmd:  nil,
		}}
	}else{
		//discard log before args.lastIncludeIndex,
		//notice that now may has been snapshot for some times, so index should be relative with rf.lastIncludeIndex
		rf.logs	= append(make([]LogEntry,0),rf.logs[args.LastIncludeIndex - rf.lastIncludeIndex:]...)
	}

	//update lastIncludeIndex & terms
	rf.lastIncludeIndex = args.LastIncludeIndex
	rf.lastIncludeTerm = args.LastIncludeTerm

	//update commit idx & last applied idx
	//saveStateWithSnapshot
	rf.persister.SaveStateAndSnapshot(rf.encodeRaftState(),args.Data)

	rf.lastApplied = Max(rf.lastApplied,rf.lastIncludeIndex)
	rf.commitIndex = Max(rf.commitIndex,rf.lastIncludeIndex)

	if rf.lastApplied > rf.lastIncludeIndex{
		return
	}
	//reset state machine using snapshot content
	applyMsg := ApplyMsg{
		CommandValid: false,
		SnapShot:     args.Data,
	}
	rf.applyCh <- applyMsg


}

func (rf *Raft) InstallSnapshot(peerIdx int) {
	rf.lock()
	//Log().Debug.Printf("install snapshot to %d",peerIdx)

	args := InstallSnapshotArgs{
		Term:             rf.currentTerm,
		LeaderId:         rf.me,
		LastIncludeIndex: rf.lastIncludeIndex,
		LastIncludeTerm:  rf.lastIncludeTerm,
		Data:             rf.persister.ReadSnapshot(),
	}
	rf.unlock()


	reply := InstallSnapshotReply{}
	ok := rf.SendInstallSnapshotRPC(peerIdx,&args,&reply)
	if ok{
		rf.lock()
		defer rf.unlock()
		if rf.currentTerm != args.Term || rf.role != Leader{
			return
		}
		if reply.Term > rf.currentTerm{
			rf.changeToFollower(reply.Term)
			return
		}
		//update match index with included index
		rf.matchIndex[peerIdx] = rf.lastIncludeIndex
		rf.nextIndex[peerIdx] = rf.lastIncludeIndex + 1
		rf.updateCommitIdx()
	}else{
		//Log().Error.Printf("install snapshot failed")
	}
}

```