---
title: MIT6.824-LAB2C
date: 2020-11-04 20:26:04
categories: 
- 分布式
- 6.824
tags: [分布式,6.824]
---

6.824 lab2c, raft之持久化状态。
<!---more--->


## 概述
看起来题目挺简单的，就是实现一个persist函数，然后在写入了需要持久化的状态的地方后调用一下persist函数就可以了。

具体需要持久化的变量在论文中也已经给出。

但是你认为这个lab就这么简单了吗？

hhhh... lab2的debug之路才刚刚开始。。。

## 持久化

实现以下persist函数，函数的使用方法也已经给出，就是new一个encoder，将要持久化的东西序列化以下。

```go
func (rf *Raft) persist() {
	// Your code here (2C).
	// Example:
	// w := new(bytes.Buffer)
	// e := labgob.NewEncoder(w)
	// e.Encode(rf.xxx)
	// e.Encode(rf.yyy)
	// data := w.Bytes()
	// rf.persister.SaveRaftState(data)

	b := new(bytes.Buffer)
	e := labgob.NewEncoder(b)
	_ = e.Encode(rf.currentTerm)
	_ = e.Encode(rf.voteFor)
	_ = e.Encode(rf.logs)
	data := b.Bytes()
	rf.persister.SaveRaftState(data)
}
```

readpersist，将已经持久化的东西反序列化出来。
```go
func (rf *Raft) readPersist(data []byte) {
	if data == nil || len(data) < 1 { // bootstrap without any state?
		return
	}
	// Your code here (2C).
	// Example:
	// r := bytes.NewBuffer(data)
	// d := labgob.NewDecoder(r)
	// var xxx
	// var yyy
	// if d.Decode(&xxx) != nil ||
	//    d.Decode(&yyy) != nil {
	//   error...
	// } else {
	//   rf.xxx = xxx
	//   rf.yyy = yyy
	// }

	b := bytes.NewBuffer(data)
	d := labgob.NewDecoder(b)

	var logs []LogEntry
	var currentTerm int
	var voteFor int

	if d.Decode(&currentTerm) != nil || d.Decode(&voteFor)!=nil || d.Decode(&logs)!=nil{
		//Log().Error.Printf("read state decoding error!!")
		return
	}
	rf.lock()
	rf.currentTerm = currentTerm
	rf.voteFor = voteFor
	rf.logs = logs
	rf.unlock()
}
```

## 2C的真正大boss：

网络不稳定，可能会延迟、RPC乱序、丢失等因素。在这种情况下会继续测试Figure8的情况。

然后可能go test 一次，没问题；但是多测几次，总会出现一些奇奇怪怪的问题。

可能是apply error：意思是两个server同一个index上的log不一致了。

### 处理乱序RPC
根据助教的文章，说到在接收到RPC回复的时候，还要检查一下自己的状态。比如是不是还是leader；比如leader发送时候的term和收到时候的term是否一致。

如果不符合，就说明由于延时等情况，收到了过期的RPC消息，这时候不应该处理任何消息。


### Figure8
Figure8要求在commit的时候，只能够commit当前term的log，所以在commit的时候需要判断一下当前的log是不是rf.currentTerm；

### 大多数的N
在commitIndex更新的时候，注意如果是偶数，取大多数的N的时候需要取靠前那个的中位数作为N；因为比如matchIndex[1,1,2,2]，N=1也可以作为commitIndex进行更新，因为1有2个 N=1 >= half of peers了

### 定时器
我原来使用了time.Timer作为定时器，后来在改了上面所有的注意事项的情况下，还是发现有过不去的情况。

翻了翻etcd的raft模块，里面他只用了一个Ticker去做定时，然后又参考了一位大神的代码，彻底放弃使用timer作为定时器。

虽然用timer作为定时器，在写事件循环的时候很方便，很直观；但是看了一些资料，golang中的timer只会响应一次，而且select选择的事件是随机的；所以每次reset timer的时候，在goroutine较多的时候可能会产生许多奇奇怪怪的问题。

参考文章如下：https://studygolang.com/articles/9289

所以timer最好使用还是用在一次性的定时器下。

而使用ticker，由于其stop可能也有一些奇奇怪怪的问题；所以最后我用的是time.After()实现选举定时；然后采用助教所说的sleep()来发送heartbeat。

助教的文章提到的是另一种方式，在rf结构体中添加一个上一次收到心跳的时间变量；然后在每次sleep唤醒后检查一下当前时间和上一次收到心跳的时间会不会超过选举时间。

而我用的time.After(electionTimeout)；然后同时select监听一个voteCh和一个heartBeatCh；只要在time.After返回信号前收到这两个channel的信息，就会重新刷新electionTimeout；这样相当于收到心跳后就reset electionTimer。

```go
go func() {
		for {
			select {
			case <-rf.stopCh:
				return
			default:
			}
			rf.lock()
			role := rf.role
			rf.unlock()
			var thisEleTimeout = rf.getRandomDuration()
			switch role {
			case Follower, Candidate:
				select {
				case <-rf.voteCh:
					//Log().Info.Printf("recv vote reset, server %d:%s",rf.me,rf.role.String())
					//thisEleTimeout = rf.getRandomDuration()
				case <- rf.heartbeatCh:
					//Log().Info.Printf("recv heartbeat reset, server %d:%s",rf.me,rf.role.String())
					//thisEleTimeout = rf.getRandomDuration()
				case <-time.After(thisEleTimeout):
					rf.lock()
					//Log().Info.Printf("ele timeout: server %d:%s",rf.me,rf.role.String())
					rf.changeToCandidate()
					rf.unlock()
				}
			case Leader:
				rf.broadCast()
				time.Sleep(heartBeatTimeout)
			}
		}
    }()
    ```



