---
title: redis设计与实现读书笔记(1)--基础数据结构
date: 2020-07-23 23:12:13
categories: redis
tags: [redis]
keywords: [redis]
---

redis系列第一篇，基本数据结构
<!---more--->

## redis基本数据结构

### 字符串
字符串SDS结构如下：
```C++
struct sdshdr{
    //记录字符串长度，也就是buf已使用的字节数
    int len;
    //buf数组未使用的字节数
    int free;
    //保存实际字符串
    char buf[];
}


```
实际上SDS也有'\0'保存在buf数组，但不计入len。

与C字符串对比：

C字符串 | SDS
:---:|:---
获取字符串长度操作为O(N)|获取字符串长度为O(1)
缓冲区溢出|修改前会计算空间，如果空间不足会对空间进行扩展，避免缓冲区溢出
修改时会造成内存重分配，需要不断地重新申请内存和释放内存|空间预分配：小于1M分配和len长度一致的空间；大于1M，分配1M。<br>惰性空间释放：不立即释放内存，将free属性修改以便后来使用
不能保存空字符，因为意味着字符串结束，所以有些编码的数据不能保存|SDS使用len代表字符串结束，不以空字符为结束标志
可使用C库函数|只可使用一部分C库函数






### 链表
```C++
struct listNode{
    //前驱结点
    struct listNode* prev;
    //后继节点
    struct listNode* next;
    //值
    void* value;
}

struct list{
    //头结点
    listNode* head;
    //尾结点
    listNode* tail;
    //链表结点数量
    unsigned long len;
    //复制函数
    void (*dup) (void *ptr);
    //释放函数
    void (*free) (void* ptr);
    //比较函数
    int (*match) (void* ptr, void* key);
}
```

用途广泛，列表键、发布订阅、慢查询、监视器都用到。

### 字典

底层实现为哈希表，一个哈希表有多个哈希表结点，一个节点保存一个键值对。

结构如下：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200724171741.png)

#### 字典相关的数据结构：
```C++

struct dictht{
    //哈希表二维数组
    dictEntry **table;
    //哈希表大小
    unsigned long size;
    //哈希表大小掩码，用于计算索引，值为size-1
    unsigned long sizemask;
    //已有哈希节点数
    unsigned long used;
}

struct dictEntry{
    //键
    void* key;
    //值
    union{
        void* val;
        uint64_t u64;
        int64_t s64;
    }v;
    //下一节点 解决哈希冲突
    struct dictEntry *next;
}

struct dict{
    //类型函数，保存了一系列用于操作特定类型的函数，比如hashfunction，复制、对比、销毁等
    dictType *type;
    //私有数据，传递给操作函数的可选参数
    void* privdata;
    //哈希表*2，rehash用到
    dictht ht[2];
    //rehash索引，记录rehash的进度
    int trehashidx

    // type privdata允许创建多态字典

}

//  相关数据依赖：  dict-->dictht-->dictEntry{key,value};

```

#### 哈希算法

计算索引值：
```C++
hash = dict->type->hashFunction(key);
index = hash & dict->ht[x].sizemask;
```

使用Murmurhash算法计算。

#### key冲突
链地址法，next指针添加结点，为方便添加，添加到表头，复杂度为O(1)

#### rehash
随着操作不断进行，哈希表的负载可能会不均衡（一些index保存很多keyvalue，一些就空），这时要对哈希表进行扩展收缩或者Rehash。

1. 分配空间：
    - 扩展：size = 第一个大于等于ht[0].used* 2 * 2^n;
    - 收缩：size = 第一个大于等于ht[0].used * 2^n;

2. 将ht[0]上面的key rehash到ht[1]上，重新计算hash和index；
3. 释放ht[0]，将ht[1]变为ht[0]，新建一个空白ht[1]；

#### 渐进式rehash
一次性rehash可能会造成redis无响应。
分index依次rehash到ht[1]中，用rehashidx记录当前进度，此时如果对字典进行操作，也会进行rehash，并且操作会在两个hash表上进行。完成时将rehashidx设为-1。



### 跳跃表
有序集合键的底层实现。

结构如下：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200724170232.png)

每一个上层可以看作是一个index，查找的时候先从最上层开始找，用next指针访问该层下一个节点，逐层往下找。

```C++
struct zskiplistNode{
    //前进指针
    zskiplistNode* backward;
    //排序用分值，分值相同，对象小的优先
    double score;
    //实际保存对象
    robj *obj;

    //level，可以看做是部分index
    struct zskiplistLevel{
        //前进指针
        struct zskiplistNode* forward;
        //跨度记录两个节点之间的距离
        unsigned int span;
    }level[];
}

struct zskiplist{
    //跳跃表头结点，尾结点
    struct zskiplistNode *head,*tail;
    //结点数量
    unsigned long length;
    //最大层数 1-32随机数
    int level;
}

```


### 整数集合

只包含整数的集合，集合键的底层实现之一，无重复。
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200724172641.png)
```C++
struct intset{
    //编码方式
    uint32_t encoding;
    //集合元素数量
    uint32_t length;
    //保存元素的数组，有序排列，声明是int8，但实际上依赖于encoding属性
    int8_t contents[];
}

```

encoding属性可以指定为16/32/64位int类型，所以可以对数组元素进行升级，提升了灵活性，节约了内存。

升级步骤：

1. 扩展空间；
2. 原数据换成新元素类型，将转换后的元素换到正确的位置上（原来16位，变成32位时，原来的第二个元素的位置也要往后挪动）。
3. 将新元素添加到原数组。

不支持降级。


### 压缩列表

压缩列表作为列表键和哈希键的底层实现之一。

当列表键列表项较少且每个是小整数值或者短字符串，用压缩列表；哈希键每个键值对都是小整数值或者短字符串，也用压缩列表。

正常数组结构如下：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200724182609.png)
上面的结构可能会造成空间浪费，因为不一定每个元素都用5字节，中间可能有空隙；

压缩列表长这样：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200724182717.png)
添加length属性，计算下一个节点的位置，压缩列表更紧凑。

redis压缩列表结构：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200724182755.png)

zlbytes:记录整个压缩列表占用的字节数，重分配、zlend用到。
zltail:记录尾结点的起始地址
zllen:压缩列表节点数
entry:列表节点
zlend:结束特殊值

redis压缩节点结构：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200724184402.png)

previous_entry_length:表示前一节点长度，通过该长度可以算出前一节点的指针地址值；如果长度小于254，为1字节；如果大于等于，为5字节，高位为0xFE表示是5字节数据，后面4字节才表示实际长度。

encoding：记录节点content类型和长度

content：保存节点值，可以是一个字节数组或者整数，类型和长度由encoding确定。

#### 连锁更新

更新前一节点的内容，使内容长度大于254字节，这时候要更改编码方式，previous_entry_length就要扩容为5字节，后面的节点也要同样扩容。这时就发生连锁更新，不断对压缩列表进行空间重分配。同样地，如果删除节点，也有可能会发生连锁更新。

最坏情况下要执行N次连锁更新，每次的复杂度为O(N),所以最坏复杂度为O(N^2);
但出现最坏情况的几率不多。

### 对象

redisObject结构如下：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200726142749.png)

type属性记录了redisObject的类型，比如REDIS_STRING/REDIS_LIST/REDIS_HASH/REDIS_SET/REDIS_ZSET

对于redis保存的对象，key总是字符串，value是type对应的类型。

encoding则对应了每种type的编码类型，比如字符串可以是REDIS_ENCODING_INT/REDIS_ENCODING_EMBSTR/REDIS_ENCODING_RAW三种编码方式。使用ENCODING属性设定对象的编码，可以提升redis的灵活性和效率，可以随着场景的使用而进行切换。

#### 字符串对象

编码方式：
1. REDIS_ENCODING_INT:
    - 字符串保存整数值
2. REDIS_ENCODING_RAW：
    - SDS保存39字节以上字符串
3. REDIS_ENCODING_EMBSTR：
    - EMBSTR也使用SDS，但是只调用一次内存分配函数分配一段连续的空间。

    EMBSTR::
    ![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200726151121.png)

    RAW::
    ![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200726151418.png)
    这里createObejct调用了一次zmalloc，sdsnewlen也调用了一次。
    ![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200726151657.png)
    ![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200726151638.png)

    另外long/double类型的也保存为字符串类型，进行相应操作可以转换成原类型然后再用字符串保存。也可以通过操作将编码升级为RAW。

#### 列表对象
列表对象编码:ziplist/linkedlist。如果列表字符串长度在65字节以下/保存的元素小于512个，用ziplist，否则用linkedlist。

#### 哈希对象
哈希对象编码：ziplist/hashtable。所有键值对的键和值得字符串长度都小于64字节，保存的键值对数量少于512个，用ziplist；否则用hashtable。

#### 集合对象
intset/hashtable。保存的数据为整数或者元素数量不超过512个，用intset，否则用hashtable，hashtable的dict的的key为保存的字符串，value为空值。

#### 有序结合对象
ziplist/skiplist：有序集合元素小于128个或者保存的元素长度小于64字节，用ziplist，否则用skiplist。

skiplist编码对象使用zset作为底层实现，一个zset同时包含一个skiplist和一个dict。

同时使用skiplist和dict是因为dict可以以O(1)的时间获取对象的score，但范围查询需要全表扫描O(N)；所以用skiplist保证范围查询是O(logN)；skiplist和dict通过指针共享相同元素的成员和分值(robj* 类指针)，不会重复保存对象。

结构大概如下：有序集合对象robj保存编码，实际数据指向底层的zset；zset根据传入的字符串，创建另一robj对象指针，分别传入dictadd和zslInsert插入，真正的数据保存在robj的void*指针里。

robj:encoding=SKIPLIST;(void*) ptr->zset;
zset:robj(key);dictadd(robj*);zslInsert*(rojb*);

#### 对象共享
对象的引用计数值除了用于内存回收外，还带有对象共享的作用；如果新建了一个同样整数值的对象，可以让keyA和keyB一起共享这个对象；每有一个键共享了这个对象，该对象的引用计数就加1 。
redis不共享包含字符串的对象，因为共享时需要先判断两个对象的值是否相同，如果保存的是字符串，验证一次的复杂度为O(n)；如果共享对象包含了N个字符串，那么验证一次的复杂度为O(N^2)。

#### 空转时长

记录上次距今访问的时间，用于LRU算法进行内存回收，释放对象。