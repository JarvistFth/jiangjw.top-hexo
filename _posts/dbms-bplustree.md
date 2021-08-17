---
title: dbms - b+树
date: 2021-08-08 23:18:05
categories: 
- 数据库
tags: [b+树,数据库]
---
b+树相关
<!---more--->

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
查找一般是在有序的范围内用log2n的算法查找，所以一般用二叉搜索树结构。

每个树的节点都可能在磁盘某一个block上，我们要做的提高效率，就是减少磁盘读取block的次数。

但是二叉搜索树可能会不平衡，这时候引入AVL树（每个结点的左右子树的高度之差的绝对值最多为1）。

但是数据量大的时候，查找次数也会增加，不停地进行磁盘IO读取block。

所以要想办法让二叉树变得“矮胖”。

这时候引入了b树。

b树是个多路平衡查找树，和b+树不一样的地方在于，他在非叶子节点中也存储数据。

b+树则是把所有数据存储在叶子节点，非叶子节点只存储索引值。

- 好处：
1. 叶子节点之间有指针连接，方便范围查询和遍历；
2. b+树的节点只存储索引值，这样可以让每个节点多存储索引值，更好地控制b+树的高度，也就更好地减少磁盘IO。
3. 查询比较稳定，都需要查询到叶子节点中。