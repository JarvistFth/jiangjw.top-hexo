---
title: dbms - buffer pool
date: 2021-08-08 23:18:05
categories: 
- 数据库
tags: [buffer-pool,数据库]
---

## buffer-pool

### 为什么需要自定义bufferpool，不适用OS的内存管理机制？

OS提供了像mmap的系统调用，可以用于内存申请；

但是db可以根据更多数据库的需求来自定义内存和外存的换入换出；具体包括：
1. 将脏页按顺序写到disk；
2. 根据具体情况自定义实现预读策略；
3. 定制化具体的缓存置换策略；
   
### file-storage
db会使用os的文件系统存储数据；和OS一样，db也会以page的形式将其作为操作的最小单位，读入内存。

### slotted-pages

page分为header和data；

header中主要包括page的元数据，包括pagesize，checksum，db-verison等。

在slotted-pages的结构下，header中还会包括slot-array，记录每个slot的大小和offset。

新增记录时：在 slot-array 中新增一条记录，记录着该记录的页内offset。
slot-array 与 data 从 page 的两端向中间生长，二者相遇时，就认为这个 page 已经满了。

删除记录时：假设删除 tuple #3，可以将 slot array 中的第三条记录删除，并将 tuple #4 及其以后的数据都都向下移动，填补 tuple #3 的空位。而这些细节对于 page 的使用者来说是透明的
处理定长和变长 tuple 数据都游刃有余。

### log structured

每次记录新的操作日志即可，增删改的操作都很快，但有得必有失，在查询场景下，就需要遍历 page 信息来生成数据才能返回查询结果。为了加快查询效率，通常会对操作日志在记录 id 上建立索引，如下图所示。


### page table
dbms会在内存中维护一个page table，负责记录每个page在内存的位置，以及dirty flag，引用计数等元数据。

当 page table 中的某 page 被引用时，会记录引用数（pin/reference），表示该 page 正在被使用，空间不够时不应该被移除；当被请求的 page 不在 page table 中时，DBMS 会先申请一个 latch（lock 的别名），表示该 entry 被占用，然后从 disk 中读取相关 page 到 buffer pool，释放 latch。

注意为了避免OS进行二次缓存，多数DBMS会用O_DIRECT告诉OS不要在OS的pagecache中缓存这些数据。

### prefetch

预读策略，减少io；

#### sequential scan

Scan Sharing 技术主要用在多个查询存在数据共用的情况。当两个查询 A, B 先后发生，B 发现自己有一部分数据与 A 共用，于是先共用 A 的 cursor，等 A 扫完后，再扫描自己还需要的其它数据。

#### Index scan
通过index进行查询，将index所在的page预先存放在buffer pool中。

### Buffer replacement policy
和OS的策略基本大同小异。

1. LRU；
2. CLOCK：每当 page 被访问时，reference bit 设置为 1
每当需要移除 page 时，从上次访问的位置开始，按顺序轮询每个 page 的 reference bit，若该 bit 为 1，则重置为 0；若该 bit 为 0，则移除该 page。
3. LRU-K：LRU-K 保存每个 page 的最后 K 次访问时间戳，利用这些时间戳来估计它们下次被访问的时间，通常 K 取 1 就能获得很好的效果。