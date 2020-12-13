---
title: MIT6-824-LAB3A
date: 2020-12-11 21:41:25
categories: 
- 分布式
- 6.824
tags: [分布式,6.824]
---
6.824 lab3A 思路和知识点整理。
<!---more--->

---
## 整体描述
lab2做了一个raft模块，可以理解为一个底层的分布式共识模块，相当于一个lib；

lab3就是在raft模块上面，实现一个client/server，提供一个key-value的存储功能，每一个peer都有一个raft模块，在每个peer的map上一起存储相同的key-value。

## 代码分析
---
Clerk对象

```go
type Clerk struct {
	servers []*labrpc.ClientEnd
	// You will have to modify this struct.
}
```

相当于一个client端，往里面添加字段。

---
KVServer对象
```go
type KVServer struct {
	mu      sync.Mutex
	me      int
	rf      *raft.Raft

	//get rf.ApplyMsg from applyCh
	applyCh chan raft.ApplyMsg
	dead    int32 // set by Kill()
	// Your definitions here.
}
```
server端，接收client发送来的消息，往里面添加字段。

---
请求的参数和回复参数

```go
package kvraft

const (
	OK          = "OK"
	ErrNoKey    = "ErrNoKey"
	WrongLeader = "WrongLeader"
)

type Msg string

const (
	PutCmd 	 = "Put"
	GetCmd	 = "Get"
	AppendCmd   = "Append"
)


type Op struct {
	// Your definitions here.
	// Field names must start with capital letters,
	// otherwise RPC will break.
	OpType string
	Key string
	Value string

}

// Put or Append
type PutAppendArgs struct {
	Key   string
	Value string
	Op    string // "Put" or "Append"
	// You'll have to add definitions here.
	// Field names must start with capital letters,
	// otherwise RPC will break.


}

type PutAppendReply struct {
	Msg Msg
}

type GetArgs struct {
	Key string
	// You'll have to add definitions here.
}

type GetReply struct {
	Msg   Msg
	Value string
}


```


## task
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20201211230545.png)

client端实现PutAppend()、Get()；
server端实现PutAppend()、Get()方法的响应方法，这些响应方法通过rf.Start()将对应的Op传入raft模块；raft在commit然后apply以后，server从applyCh读这个Op，然后返回消息给client。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20201211231051.png)
client请求的幂等性（server不会重复处理client同一请求），client的发起的RPC调用失败或超时的话，可能这个server不是leader，需要重复往另一个server发请求。

## client端代码

几个要点：
1. 幂等性，为每个请求加一个RequestID，server端判断，如果这个RequestID大于Op的RequestID的时候，才进行处理；
2. 找不到leader的时候，需要不断的轮询raft集群，直到找到leader，为了避免不必要的轮询，每个client可以记住自己的lastServerID；
3. 一个client的时候，上述代码就可以了；但是如果是多个client，还需要注意为每个client回复对应的请求，所以还需要一个clientid来标识。

```go
package kvraft

import (
	"labrpc"
	"time"
)
import "crypto/rand"
import "math/big"


type Clerk struct {
	servers []*labrpc.ClientEnd
	// You will have to modify this struct.

	lastServer int

	ClientId int64
	RequestId int64


}

func nrand() int64 {
	max := big.NewInt(int64(1) << 62)
	bigx, _ := rand.Int(rand.Reader, max)
	x := bigx.Int64()
	return x
}

func MakeClerk(servers []*labrpc.ClientEnd) *Clerk {
	ck := new(Clerk)
	ck.servers = servers
	// You'll have to add code here.

	ck.ClientId = nrand()
	ck.RequestId  = 0
	return ck
}

//
// fetch the current value for a key.
// returns "" if the key does not exist.
// keeps trying forever in the face of all other errors.
//
// you can send an RPC with code like this:
// ok := ck.servers[i].Call("KVServer.Get", &args, &reply)
//
// the types of args and reply (including whether they are pointers)
// must match the declared types of the RPC handler function's
// arguments. and reply must be passed as a pointer.
//
func (ck *Clerk) Get(key string) string {

	// You will have to modify this function.
	args := GetArgs{Key: key,ClientID: ck.ClientId,RequestID: ck.RequestId}
	reply := GetReply{}
	i:= ck.lastServer
	ck.RequestId++
	for{
		//Log().Info.Printf("client %d , send get rpc to server %d",ck.ClientId,i)
		ok := ck.servers[i].Call("KVServer.Get",&args,&reply)
		if ok && reply.Msg == OK{
				ck.lastServer = i
				return reply.Value

		}
		i = (i+1) % len(ck.servers)
		time.Sleep(time.Duration(100)*time.Millisecond)
	}
}

//
// shared by Put and Append.
//
// you can send an RPC with code like this:
// ok := ck.servers[i].Call("KVServer.PutAppend", &args, &reply)
//
// the types of args and reply (including whether they are pointers)
// must match the declared types of the RPC handler function's
// arguments. and reply must be passed as a pointer.
//
func (ck *Clerk) PutAppend(key string, value string, op string) {
	// You will have to modify this function.

	args := PutAppendArgs{
		Key:   key,
		Value: value,
		Op:    op,
		ClientID: ck.ClientId,
		RequestID: ck.RequestId,
	}

	ck.RequestId++

	reply := PutAppendReply{}
	i := ck.lastServer
	for{
		//Log().Info.Printf("client %d , send put rpc to server %d",ck.ClientId,i)
		ok := ck.servers[i].Call("KVServer.PutAppend",&args,&reply)
		if ok && reply.Msg == OK{
			ck.lastServer = i
			return
		}
		i = (i+1) % len(ck.servers)
		time.Sleep(time.Duration(100)*time.Millisecond)
	}
}

func (ck *Clerk) Put(key string, value string) {
	ck.PutAppend(key, value, "Put")
}
func (ck *Clerk) Append(key string, value string) {
	ck.PutAppend(key, value, "Append")
}

```

## server端代码

注意点：
1. 多client并发的情况下，应该记住每个client的request，对每个client回复对应的消息；所以用一个map记录了clientid对应的requestid；
2. 幂等性判断，只有大于map中的requestid才去操作server里面的key/value；读操作由于是幂等，我就没有在applyCh那里判断了。
3. server端的PutAppend()或者Get()都是通过start将Op送入raft模块；然后从applyCh听消息；所以判断幂等性的操作应该在applyCh返回消息以后进行判断，不应该在进入PutAppend和Get就判断
【为什么呢？】因为这时候client发起的命令不一定会被raft commit，如果这次发送的request没有被commit，那么server会返回一个timeout结果给client，这时候其实是不应该记录这个requestid在map里面的，因为这时候request不成功，应该要重复发送，下次继续处理这个request。
4. 同样的，从applyCh收raft回复的Op，多client并发的情况下要分清楚这个Op是哪个client送进去的，所以要更新Op的字段，添加clientID和requestID；【区分措施】收到raft送来的Op以后，用一个map记录每个clientid对应的channel，然后获取Op.client对应的channel，往这个channel发Op，然后PutAppend或者Get听这个channel，如果接收到消息，再对比一下和之前发送的Op是不是一致【乱序的情况下可能会存在同一个client但是返回的Op不一致的情况？】，如果是就回复给client。
5. 在PutAppend和Get方法中监听clientid对应channel的时候，如果这时候leader是在一个少数网络分区里面，就会一致没有办法commit，对应applyCh也无法返回消息，所以这时候clientid对应channel也读不到消息，所以要在这里添加一个超时的逻辑。


```go
package kvraft

import (
	"bytes"
	"labgob"
	"labrpc"
	"raft"
	_ "strconv"
	"sync"
	"sync/atomic"
	"time"
)


type KVServer struct {
	mu      sync.Mutex
	me      int
	rf      *raft.Raft

	//get rf.ApplyMsg from applyCh
	applyCh chan raft.ApplyMsg
	dead    int32 // set by Kill()

	db          map[string]string

	OpMap 		map[int]chan Op
	ReqMap		map[int64]int64

	maxraftstate int // snapshot if log grows this big
	persister *raft.Persister
	killCh chan struct{}
	// Your definitions here.
}


func (kv *KVServer) Get(args *GetArgs, reply *GetReply) {
	// Your code here.
	//Log().Info.Printf("get args from client")


	op := Op{
		OpType: GetCmd,
		Key:    args.Key,
		Value: "",
		ClientId: args.ClientID,
		RequestId: args.RequestID,
	}
	index,_,isLeader := kv.rf.Start(op)
	if !isLeader{
		reply.Value = ""
		reply.Msg = WrongLeader
		return
	}
	ch := kv.getOpIndexCh(index)
	//newOp := <- ch

	newOp := Op{}
	select {
	case op := <-ch:
		//Log().Info.Printf("get op")
		newOp = op
	case <- time.After(time.Duration(600)*time.Millisecond):
		//Log().Info.Printf("get timeout")
	}

	if newOp.Equal(op){
		kv.mu.Lock()
		_,found := kv.db[op.Key]
		kv.mu.Unlock()
		if !found {
			reply.Value = ""
			reply.Msg = ErrNoKey
			return
		}else{
			kv.mu.Lock()
			reply.Value = kv.db[op.Key]
			kv.mu.Unlock()
			reply.Msg = OK
			return
		}
	}else{
		reply.Msg = WrongLeader
		return
	}

}

func (kv *KVServer) PutAppend(args *PutAppendArgs, reply *PutAppendReply) {
	// Your code here.
	//Log().Debug.Printf("recv put-append args from client %d", args.ClientID)

	op := Op{
		OpType: args.Op,
		Key:    args.Key,
		Value:  args.Value,
		ClientId:  args.ClientID,
		RequestId: args.RequestID,

	}
	index,_,isLeader := kv.rf.Start(op)
	if !isLeader{
		reply.Msg = WrongLeader
		return
	}

	ch := kv.getOpIndexCh(index)
	//newOp := <- ch
	//newOp := getOpOrTimeout(ch)
	//Log().Debug.Printf("get ch with index:%d",index)
	newOp := Op{}
	select {
	case op := <-ch:
		//Log().Info.Printf("getop")
		newOp = op
	case <- time.After(time.Duration(600)*time.Millisecond):
		//Log().Info.Printf("put timeout")
	}

	if newOp.Equal(op){
		reply.Msg = OK
		return
	}else{
		reply.Msg = WrongLeader
		return
	}
}

//
// the tester calls Kill() when a KVServer instance won't
// be needed again. for your convenience, we supply
// code to set rf.dead (without needing a lock),
// and a killed() method to test rf.dead in
// long-running loops. you can also add your own
// code to Kill(). you're not required to do anything
// about this, but it may be convenient (for example)
// to suppress debug output from a Kill()ed instance.
//
func (kv *KVServer) Kill() {
	atomic.StoreInt32(&kv.dead, 1)
	kv.rf.Kill()
	// Your code here, if desired.

}

func (kv *KVServer) killed() bool {
	z := atomic.LoadInt32(&kv.dead)
	return z == 1
}

func (kv *KVServer) getOpIndexCh(index int) chan Op{
	kv.mu.Lock()
	defer kv.mu.Unlock()
	 _,found := kv.OpMap[index]
	 if !found{
	 	kv.OpMap[index] = make(chan Op, 1)
	 }
	 return kv.OpMap[index]
}

func sendCh(ch chan Op, op Op) {
	select {
	case <-ch:
	default:
	}
	ch <- op
}

func (kv *KVServer) takeSnapshotOnDB(index int) {
	b := new(bytes.Buffer)
	e := labgob.NewEncoder(b)

	kv.mu.Lock()
	_ = e.Encode(kv.db)
	_ = e.Encode(kv.ReqMap)
	kv.mu.Unlock()
	kv.rf.TakeSnapshot(index,b.Bytes())
}

func (kv *KVServer) restoreSnapshot(snapshot []byte) {
	if snapshot == nil || len(snapshot) < 1{
		return
	}

	kv.mu.Lock()
	defer kv.mu.Unlock()

	b := bytes.NewBuffer(snapshot)
	d := labgob.NewDecoder(b)

	var db map[string]string
	var ReqMap map[int64]int64

	if d.Decode(&db) != nil || d.Decode(&ReqMap) != nil{
		Log().Error.Printf("could not decode db && client-req map!!")
	}else{
		kv.db = db
		kv.ReqMap = ReqMap
	}
}

func (kv *KVServer) needSnapshot() bool {
	kv.mu.Lock()
	defer kv.mu.Unlock()
	//Log().Debug.Printf("raftStateSize:%d, maxRaftState:%d",kv.persister.RaftStateSize(),kv.maxraftstate)
	return kv.maxraftstate > 0 &&  kv.maxraftstate - kv.persister.RaftStateSize() < kv.maxraftstate/10
}


//
// servers[] contains the ports of the set of
// servers that will cooperate via Raft to
// form the fault-tolerant key/value service.
// me is the index of the current server in servers[].
// the k/v server should store snapshots through the underlying Raft
// implementation, which should call persister.SaveStateAndSnapshot() to
// atomically save the Raft state along with the snapshot.
// the k/v server should snapshot when Raft's saved state exceeds maxraftstate bytes,
// in order to allow Raft to garbage-collect its log. if maxraftstate is -1,
// you don't need to snapshot.
// StartKVServer() must return quickly, so it should start goroutines
// for any long-running work.
//
func StartKVServer(servers []*labrpc.ClientEnd, me int, persister *raft.Persister, maxraftstate int) *KVServer {
	// call labgob.Register on structures you want
	// Go's RPC library to marshall/unmarshall.
	labgob.Register(Op{})

	kv := new(KVServer)
	kv.me = me
	kv.maxraftstate = maxraftstate
	kv.persister = persister

	// You may need initialization code here.

	kv.applyCh = make(chan raft.ApplyMsg)
	kv.killCh = make(chan struct{})

	kv.rf = raft.Make(servers, me, persister, kv.applyCh)

	kv.db = make(map[string]string)
	kv.OpMap = make(map[int]chan Op)
	kv.ReqMap = make(map[int64]int64)

	kv.restoreSnapshot(kv.persister.ReadSnapshot())

	// You may need initialization code here.
	go func() {
		for {
			select {
			case <- kv.killCh:
				return
			case msg := <- kv.applyCh:
				if !msg.CommandValid{
					Log().Info.Printf("restore snapshot")
					kv.restoreSnapshot(msg.SnapShot)
					continue
				}
				op := msg.Command.(Op)
				Log().Debug.Printf("get msg from apply ch, idx: %d, value:%s", msg.CommandIndex,op.Value)
				clientid := op.ClientId

				kv.mu.Lock()
				reqNum,found := kv.ReqMap[clientid]

				//kv.db as state machine, dont use same cmd on state machine
				if !found || op.RequestId > reqNum{
					switch op.OpType {
					case PutCmd:
						kv.db[op.Key] = op.Value
					case AppendCmd:
						kv.db[op.Key] += op.Value
					}
					kv.ReqMap[op.ClientId] = op.RequestId
				}
				kv.mu.Unlock()
				index := msg.CommandIndex
				ch := kv.getOpIndexCh(index)
				if kv.needSnapshot(){
					//Log().Debug.Printf("before log size:%d",kv.persister.RaftStateSize())
					//like redis db file saveState, new goroutine to take snapshot
					go kv.takeSnapshotOnDB(index)
					//kv.takeSnapshotOnDB(index)
					//Log().Debug.Printf("after log size:%d",kv.persister.RaftStateSize())
				}
				//Log().Debug.Printf("send ch with index:%d op:%s",index, op.Value)
				sendCh(ch,op)
			}

		}
	}()
	Log().Info.Printf("server start")
	return kv
}

```


