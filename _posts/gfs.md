---
title: 论文 - GFS
date: 2020-12-31 19:55:44
categories: 
- 论文
tags: [论文, 6.824 ,2020]
keywords: [论文,2020]
---

重读gfs，看看有没有新感悟。

## Abstract

1. scalable 分布式文件系统， 用于大型分布式的数据密集型应用。
2. 在廉价设备上提供了FT（Fault-tolerance）;
3. 通过观察应用的workload和技术环境，重新做了设计选择。

## Introduction

GFS和以前的分布式文件系统一样，有着一些共同的设计目标，包括：performance（性能）、scalability（可扩展性）、reliability（可靠性）、availability（可用性）。

performance自不用说，scalability指的应该是随着数据量的增大是否可以通过简单的添加机器实现水平扩展；reliability应该说的是防止数据丢失；availability是指client发出请求后在一定时间内能够收到回应。

观察经验，和以往的分布式文件系统不一样的点在于：
1. 系统里的组件出现故障是常态，不应当成是exception来设计。因为虽然可能一个部件的故障概率不大，但是随着系统的扩展，可能会拥有越来越多的部件，这样对于系统来说里面某一个部件出现的故障概率就不小了。
2. 大文件是常态。将大文件分成几十kb的块来进行存储，对他们进行管理是很麻烦的事情。所以IO操作和块大小需要重新考虑设计。
3. 大多数文件的更改是通过追加写新数据实现的，而不是通过覆盖现有数据。这种模式下，追加写成为了性能优化和Atomic需要保证的重点。
4. 将应用和文件系统API共同灵活设计有助于系统设计。比如GFS简化了一致性模型以简化文件系统，减轻对应用的负担；为客户端提供了Atomic追加写操作，不需要客户端之间再去同步。

## Overview

### 假定条件

1. 这个系统是由许多经常出故障的商品组成的。所以它必须不断地监控自己，在日常中检测、容忍组件故障，并快速从中恢复。
2. 系统存储少量的大文件。可能有几百万个文件，每个文件的大小通常是100mb或更大。以gb为量级的文件是常见的情况，应该有效地管理。必须支持小文件，但我们不需要为它们进行优化。
3. workload主要分两种读情况：
   - large streaming reads:单个读操作可能是1m/s以上；同一客户端连续读取文件的某一连续区域。
   - small random reads ：通常以任意偏移量读取几kb。对性能敏感的应用程序会对它们的小读取进行批处理和排序，防止读取offset变来变去。（可以类比磁盘的随机读）
4. workload还具有许多大量的、顺序的写入，将数据追加到文件中。文件一旦写入，就很少再被修改。需要支持在文件中的任意位置进行小量写操作，但不一定要追求高效。
5. 系统必须高效地为并发追加写到同一文件的多个client实现定义良好的语义。因为可能有多个生产者同时追加写入到同一文件中，实现atomic和同步的开销要尽可能小。文件可以稍后读取，也可以让多个消费者同时读取。
6. 高持续带宽比低延迟更重要。大多数目标应用程序都注重以高速率批量处理数据，而很少有对单个读或写有严格的响应时间要求。（对带宽要求敏感，对延迟不敏感，和实时计算相反）

### interface

1. 类POSIX接口，以目录、path、name形式组织，支持create、delete、open、close等类POSIX操作；
2. 支持snapshot和追加写操作。snapshot支持低成本创建文件/目录的副本，追加写则允许多客户端并发往一个文件中写入数据并同时保证操作的atomic。提供这样的API可以减少客户端之间额外的锁的使用。

### Architecture


![](https://jaroffertree-1255739634.cos.ap-guangzhou.myqcloud.com/20211217152511.png)

- 一个master和若干个chunkserver。通过client与master、chunkserver交互。
- 每个file划分成若干个固定大小的chunks，每个chunk由不可变的uint64的handle标识，由master分配。每个chunkserver将chunks以file的形式存储在磁盘上，读写由handle和offset决定数据范围。
- 为了保证reliability，需要有多副本存储，。

- master存储系统的metadata，包括namespace、access control information、文件到chunks的映射、chunks所在的位置等。
- master还控制系统的活动，包括lease机制管理、orphaned chunks的GC，以及chunkserver之间的chunk migration。所以master会定期与chunkserver发送heartbeat，收集状态信息以及发送指令等。
- client端实现了FS-API，但是需要和master和chunkserver通信来实现读写文件。
- client和chunkserver都不会缓存文件数据。client不缓存文件数据可以减少缓存一致性的问题，但是client会缓存metadata。chunkserver不需要额外缓存，因为linux自己就会将常用的数据缓存在内存中。

### metadata
包括三类：file和chunk的namespace，file到chunk的映射；每个chunk的replica的位置。

所有的metadata都保存在内存中，前两类通过log进行持久化存储。对于每个chunk的replica的位置，在master启动以及chunkserver加入集群的时候进行初始化。然后master就可以通过定期的heartbeat更新chunkserver的状态信息，从而更新每个replica的位置。

通过log重放可以恢复master的状态；但是为了log过多初始化时间过长，引入checkpoint。创建checkpoint后，从checkpoint开始重放log恢复master。

### 一致性模型


### 其他
1. GFS用了单一master，简化设计。（master崩了就g了）
2. chunksize用了64MB，1）用于减少client与master、chunkserver的交互；2）让client更大几率在一个TCP连接中对chunk进行操作；3）减少master中存储的metadata的量。缺点就是小文件可能用了更大的chunk，可能会造成热点访问。







