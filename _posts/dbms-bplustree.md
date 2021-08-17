---
title: dbms - b+树
date: 2021-08-08 23:18:05
categories: 
- 数据库
tags: [b+树,数据库]
---

# B+树

B Tree 与 B+ Tree 最主要的区别就是 B+ Tree 的所有 key 都存储在 leaf nodes，而 B Tree 的 key 会存在所有节点中。

B+ Tree 是一种自平衡树，它将数据有序地存储，且在 search、sequential access、insertions 以及 deletions 操作的复杂度上都满足  ，其中 sequential access 的最终复杂度还与所需数据总量有关。

## M路B+树

1. 每个节点最多存储M个key，有M+1个children；
2. B+树是严格平衡二叉树，所有leaf-node深度一样；
3. 除了root节点外，其他节点都是半满状态，即每个节点的key编号在[M/2-1,M-1]；
4. 每个中间节点有k个key就有k+1个children；
5. leaf-node有双向指针链接成为一个双向链表，从而为范围查询提供高效支持。

## B+树 Operation：
### Insert
1. 找到对应的 leaf node，L
2. 将 key/value pair 按顺序插入到 L 中；
3. 如果 L 还有足够的空间，操作结束；如果空间不足，则需要将 L 分裂成两个节点，同时在 parent node 上新增 entry；
4. 若 parent node 也空间不足，则递归地分裂，直到 root node 为止。

### Delete
1. 从 root 开始，找到目标 entry 所处的 leaf node, L
2. 删除该 entry
3. 如果 L 仍然至少处于半满状态，则操作结束；否则先尝试从 siblings节点 那里拆借 entries，如果失败，则将 L 与相应的 sibling节点 合并
4. 如果合并发生了，则可能需要递归地删除 parent node 中的 entry


## concurrency controll
两种情况：
1. 多个线程同时修改同一个 node
2. 一个线程正在遍历 B+ Tree 的同时，另一个线程正在 splits/merges nodes

latch用rw-latch，也就是有两种mode；(其实就是读写锁)

* read-latch:支持多个线程读同一份数据；
* write-latch:只要有一个线程读或者写，都不允许获取；


### path latching algorithm

B+树是树形结构，从root往下访问，所以可以考虑从路径开始入手加锁；

基本思想：（获取的都是写锁）
1. 获取parent的latch；
2. 获取children的latch；
3. 如果安全，释放parent latch；
   * 安全的定义：发生更新操作时，该节点不会发生split或者merge。即：
   * 插入元素时，节点未满；
   * 删除元素时，节点超过满的一半；


问题：每次更新操作在每个路径上都要获取写锁，可能会造成瓶颈；实际上每个node大小为一个page，一个page存储很多个key，因此对每个leaf-node来说更新操作的出现频率并不算高；

优化：采用乐观锁的思想，每次都假设leaf-node是安全的，即更新操作仅仅会引起leaf-node变化；在查询路径上一路获取、释放 read-latch，到达 leaf-node 时，若操作不会引起 split/merge 发生，则只需要在 leaf-node 上获取 write-latch 然后更新数据，释放 write-latch 即可；若操作会引起 split/merge 发生，则重新执行一遍，此时在查询路径上一路获取、释放 write latch，即原始方案。




### horizontal scan latching algorithm

刚才的加锁都是沿着路径加锁的，但是在leaf-node上我们可能需要进行范围访问，这个时候需要水平左右加锁。

在进行左右加锁的时候，两个线程可能会因为方向不一，导致死锁。当遇到横向扫描无法获取下一个节点的 latch 时，该线程将释放 latch 后自杀。这种策略逻辑简单，尽管有理论上的优化空间，但在实践中是常见的避免死锁的方式。

### 延迟更新优化

每当 leaf node 溢出时，我们都需要更新至少 3 个节点：
1. 即将被拆分的 leaf node
2. 新的 leaf node
3. parent node

修改的成本较高，因此 B-link Tree 提出了一种优化策略：每当 leaf node 溢出时，只是标记一下而暂时不更新 parent-node，等下一次有别的线程获取 parnet-node 的 write-latch 时，一并修改。

## B+树优点：
### Transaction
transaction 是 DBMS 状态变化的基本单位，一个 transaction 只能有执行和未执行两个状态，不存在部分执行的状态.


#### ACID
* A: 原子性：要么全部执行，要么全部不执行；
* C：一致性：执行结果符合客观事实；
* I：隔离性：不同事务之间不会互相影响；
* D：持久性：执行成功后即使断掉等也不会丢失数据。

#### 隔离级别
|级别|含义|问题
|---|---|---|
读未提交|可以读到其他事务没有提交的数据|脏读
读已提交|只可以读到其他事务已经提交的数据|事务过程中发生了两次读取，两次读取的结果不一致，也就是不可重复读|、
可重复读|MVCC决定了这次事务每次重复读到的数据都是一样的|幻读，插入删除等范围查询的时候可能会出现突然多出来一些数据的情况
串行化|间隙锁保证事务执行过程中不会出现幻读|效率较低


#### 如何保证原子性：

方法：log先行，没执行一个命令都写入一个undo-log，然后如果transaction abort了，就执行undo-log中的命令。


#### 如何保证隔离性？
可以通过对比每个事务的读写集来看是否存在冲突；但是该方法需要知道每个事务的具体执行流程，实际上用不上。因此需要保证每个事务读写的数据不能冲突。简单的想法就是加锁。

### Two-Phase-Locking（2PL）

dbms中有专门的lock manager，负责管理dbms中的lock，由它来向事务中的数据进行加锁和解锁。
lock-manager内部维护着一个lock table，记录着锁的分配信息，来决定是否给予锁。

- 为什么需要2PL呢？

  因为仅仅在写入或者访问数据的时候加锁并不能保证事务的正确性。

2PL：growing、shrinking

* growing：事务可以获取锁，lock-manager决定同意或者拒绝。
* shrinking：事务只能释放锁，不能重新获得锁。

- 问题：死锁，两个事务互相持有锁，都等待对方释放：
  
  - 死锁检测：有向图，定期查看图是否有环，如果有环就想办法怎么打破环，选择一个事务终止其执行，打破环。
  - 死锁预防：在事务尝试获取其它事务持有的锁时直接决定是否需要将其中一个事务中止。
    - Old Waits for Young：如果 requesting txn 优先级比 holding txn 更高则等待后者释放锁；更低则自行中止
    - Young Waits for Old：如果 requesting txn 优先级比 holding txn 更高则后者自行中止释放锁，让前者获取锁，否则 requesting txn 等待 holding txn 释放锁


### TimeStamp Ordering Concurrency Control
核心思想：利用时间戳来决定事务之间的等价执行顺序：如果TS(Ti) < TS(Tj)  ，那么数据库必须保证实际的 schedule 与先执行Ti，后执行Tj的结果等价。

#### Basic TimeStamp Ordering

每条数据都会携带它最后一次写的时间和最后一次读的时间。
* 如果一个事务发生在它最后一次写的时间之前，即它尝试读取未来的数据，就中止事务；
* 如果一个事务发生在它最后一次写的时间之后，符合规范；
  
### OCC（Optimistic Concurrency Control）
所有读取的数据都在复制一份到私有空间，所有修改都在所有空间中进行。（读写集）

OCC分为三个阶段
1. Read Phase：追踪、记录每个事务的读写集合，存储到私有空间中；
2. Validation Phase：事务提交时，检查读写集是否冲突；
3. Write Phase：如果通过Validation，合并数据；否则中止事务。

#### 读写冲突检测：

* 后向检测：检测待提交事务和已提交事务的读写集合是否存在交集；
* 前向检测：TS(Ti) < TS(Tj)；
  1. 事务Ti在事务Tj开始之前已经完成 OCC 的所有 3 个阶段
  2. Ti在Tj的Write Phase开始前就提交，则需要满足：WriteSet(Ti) ∩ ReadSet(Tj) == ∅
  3. Ti在Tj结束自己的Read Phase之前结束Read Phase，需要满足：WriteSet(Ti) ∩ ReadSet(Tj) == ∅ && WriteSet(Ti) ∩ WriteSet(Tj) == ∅


### MVCC
主要思想是在DBMS内部维持单个数据的多个物理版本，从而实现读写互不阻塞。

#### 实现组件
1. Concurrency Control Protocol
2. Version Storage
3. Garbage Collection
4. Index Management
   

##### Concurrency Control Protocol
可以选择TimeStamp Orderring/ 2PL / Optimistic Concurrency Control

##### Version Storage
三种方式存储不同版本的数据

1. Append-Only Storage
   新版本数据通过追加的方式存储到同一张表中；每次更新时，就往表上追加一个新的版本记录，并在旧版本的数据上增加一个指针指向新版本。
2. Time-Travel Storage
   老版本被单独复制到一张表中；每当更新数据时，就把当前版本复制到那张单独的表中，并更新指针。（有点类似区块链）
3. Delta Storage
   只存储被修改的字段，复制到一张单独的delta record space中。（有点类似redis的aof，每次都是一个修改的命令）

##### Garbage Collection
随着时间的推移，DBMS 中数据的旧版本可能不再会被用到，如：
- 已经没有活跃的事务需要看到该版本
- 该版本是被一个已经中止的事务创建

这个时候DBMS就需要找出这些可以回收的数据的物理版本，然后进行回收，也就是GC。

* tuple-level gc：
    1. 通过后台线程定期查询每条数据的不同版本，比较它的结束时间和当前活跃事务的最小时间戳。

    2. 或者在查询的时候，顺便删除不再使用的版本数据。
* transaction-level gc：
  让每个事务都保存着它的读写数据集合 (read/write set)，当 DBMS 决定什么时候这个事务创建的各版本数据可以被回收时，就按照集合内部的数据处理即可。

##### Index ManageMent

对于primary key index，直接指向version chain的首部；

而对于二级索引，则有两种方式。
1. 逻辑指针，二级索引指向primary key或者 tuple id；
2. 物理指针，指向version chain头部。