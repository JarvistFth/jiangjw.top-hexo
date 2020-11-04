---
title: MIT6.824-LAB2B
date: 2020-11-01 19:47:11
categories: 
- 分布式
- 6.824
tags: [分布式,6.824]
---

6.824 lab2b, raft之日志复制。
<!---more--->
## 日志复制

当选出leader后，leader开始向follower发送日志/心跳。在论文中，心跳和日志其实是同一个东西，心跳可以看作是没有携带log的appendEntries。

我们要完成的，就是一个snedAppendEntry的发送方RPC以及一个AppendEntry的RPC handler。

具体RPC的参数，在论文中也已经给出。这里就不再详述。

## sendAppendEntry RPC
这里是用于leader向follower发送RPC；构造好参数后，用goroutine发送RPC。

看RPC Handler返回的结果；
如果是reply.Success == true：
1. 证明此时follower接收了leader的日志，更新nextIndex[i]和matchIndex[i]；
2. 更新自己的commitIndex；
如果reply.Success == false：
1. 证明这时候leader 往follower 发送的日志并不一致，根据论文所述，需要回退nextIndex，并在下次AppendEntry的时候继续发送。

```go

func (rf *Raft) sendAppendEntriesRPC(peerIdx int) {
	reply := AppendEntriesReply{}

	rf.lock()

	if rf.role != Leader {
		rf.unlock()
		return
	}

	prevLogIndex := rf.nextIndex[peerIdx] - 1

	args := AppendEntriesArgs{
		Term:            rf.currentTerm,
		LeaderId:        rf.me,
		PrevLogIndex:    prevLogIndex,
		PrevLogTerm:     rf.getTerm(prevLogIndex),
		Entries:         nil,
		LeaderCommitIdx: rf.commitIndex,
	}
	args.Entries = make([]LogEntry, len(rf.logs[prevLogIndex+1:]))
	copy(args.Entries, rf.logs[prevLogIndex+1:])
	rf.unlock()
	//Log().Debug.Printf("prevLogIdx:%d, argLogLen:%d",prevLogIndex,len(args.Entries))

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
				rf.nextIndex[peerIdx] = reply.ConflictIndex
			}
		}
	}
}
```

## AppendEntry RPC Handler

这里是用于follower响应leader发来的请求，相关的参数论文中也已经给出。

1. 如果接收到leader的term 大于自己的term；将自己变为follower；
2. 否则如果leader发来的term小于自己的term，就丢弃（认为该信息过期）；
3. 日志限制：如果当前日志更短，不符合leader的prevLogIndex；如果一个follower在日志中有prevLogIndex，但是该index上日志的term不匹配，返回false。
4. 如果通过了日志限制判断，这时候prevLogIndex上的日志的term是相同了；但是根据论文提示，后面可能有个日志和新的日志有冲突（same index different term），这时候要找到新发来的logs冲突的index；然后保留follower的[0:conflictidx+1)日志，从args.logs[conflictidx:)开始拼接leader发来的新日志。
5. 更新commitIndex为leaderCommit和自身最后一个日志index的最小值。（因为leader可能commit了很多的日志了已经，超出自身日志长度）

```Go
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
	if rf.getLastLogIndex() < args.PrevLogIndex {
		//Log().Warning.Printf("missing log")
		reply.ConflictIndex = rf.getLastLogIndex() + 1
		reply.ConflictTerm = -1
		return
	}

	//如果一个follower在日志中有prevLogIndex，但是term不匹配，
	//它应该返回conflictTerm = log[prevLogIndex].Term，
	//并且查询它的日志以找到第一个term等于conflictTerm的条目的index。
	if rf.getTerm(args.PrevLogIndex) != args.PrevLogTerm{
		//Log().Warning.Printf("entry at prevLogIndex whose term does not matches prevLogTerm")

		//contain logs that leader doesn't have
		reply.ConflictTerm = rf.getTerm(args.PrevLogIndex)
		conflictidx := args.PrevLogIndex

		for rf.getTerm(conflictidx - 1) == reply.ConflictTerm{
			conflictidx --
		}
		reply.ConflictIndex = conflictidx
		return
	}

	//If an existing entry conflicts with a new one (same index
	//but different terms), delete the existing entry and all that
	//follow it (§5.3) Figure 7 (f)
	//now has the same previdx and logs[previdx].term, but its term may be different


	//Append any new entries not already in the log

	oldLeaderUnCommitedIdx := -1
	//Log().Info.Printf(args.String())

	for i,e:=range args.Entries{
		//i start from 0
		//append new || same index but different term
		if rf.getLastLogIndex() < args.PrevLogIndex+1+i || e.Term != rf.logs[(args.PrevLogIndex+i+1)].Term {
			oldLeaderUnCommitedIdx = i
			break
		}
	}
	
	// 1 1 4 4 5
	// 1 1 4 5 5
	if oldLeaderUnCommitedIdx != -1{
		rf.logs = rf.logs[:args.PrevLogIndex + oldLeaderUnCommitedIdx+1]
		rf.logs = append(rf.logs,args.Entries[oldLeaderUnCommitedIdx:]...)
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
```

## 一些优化
整体的思路如同上面所说，但是我的代码和我说的有一点不太一样；做了一些优化，Morris教授在视频中也提到。

### fast backup：

如果follower落后了leader很多日志，将leader的nextindex不断减1就显得有点笨重了。教授在课上提到了利用Xterm和Xindex加快回退nextIndex的过程，但是说得比较快比较难理解。在助教的文章中则比较清晰的提到了。

1. 如果一个follower在日志中没有prevLogIndex，它应该返回conflictIndex=len(log)和conflictTerm = None。

2. 如果一个follower在日志中有prevLogIndex，但是term不匹配，它应该返回conflictTerm = log[prevLogIndex].Term，并且查询它的日志以找到第一个term等于conflictTerm的条目的index。

### 更新commitIndex
在更新完commitIndex以后，我们上层的应用可以将他apply到状态机里了。在lab2里面，就是想applyCh发送一个applyMsg。

在这里我们可以使用两种方式：
1. 在make的时候就创建一个daemon goroutine，每隔一段时间监视一下applyIndex和commitIndex的值，如果commitIndex更大，就将他apply到状态机中；
2. 在每次更新commitIndex的时候都调用一次发送applyMsg的函数。

我使用的是第二种方式，由于这时候发送applyMsg也是在锁里面，所以效率可能不会太高，可能用第一种方式会好点，但是具体还没有实测。


