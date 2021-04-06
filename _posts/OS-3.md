---
title: 操作系统 - 进程间通信与线程同步
date: 2021-02-24 18:44:28
categories: 
- OS
- 进程间通信与线程同步
tags: [OS,进程间通信,线程同步]
keywords: [OS,进程间通信,线程同步]

---

进程间通信方式，线程同步方式总结
<!---more--->

## 进程间通信

### 信号

原理是：注册信号对应的handler，收到信号时，系统调用将应用程序堆栈修改为handler入口，执行handler函数，再将原执行地址作为handler函数的返回地址.

linux中对应的是signal函数。
```C
void (*signal(int signo,void (*func)(int)))(int);
```

第一个int参数是要捕获的信号（整形常量），第二个参数是一个函数指针（处理函数），该函数指针指向的函数返回值是void，参数是int。


### 管道

#### 无名管道

linux管道只能在有亲缘关系的进程间使用。

```C
int pipe(int fd[2]){

}
```
通过pipe()函数创建管道，fd[0]读，fd[1]写。 创建的是无名管道；

#### 命名管道

FIFO，不相关进程也能通过FIFO管道交换数据。

```C
int makefifo(const char* path, mode_t mode){

}

int mkfifoat(int fd,const char *path,mode_t mode){

}

```




### 消息队列

消息队列，正如其名称一样，是消息的链表形式，它由内核存储维护。

msgget函数创建一个队列或者打开一个现有队列，msgsnd将新数据添加到消息末尾，每个消息包含一个正的长整形字段、一个非负的长度以及实际数据字节数。msgrcv从队列中获得消息。

### 共享内存

共享内存，顾名思义，就是划分一个共享的内存地址空间给两个进程使用。

linux中有mmap()和shmget()两种。 mmap如果宕机了还有文件留存，shmget则不会。

#### mmap

mmap的原理是，在虚拟内存上获取一段地址，在磁盘上创建一个普通文件，虚拟地址作为普通文件映射到的地址空间，然后进程间就可以像通过访问普通文件fd那样对fd进行操作。

#### shmget

```C
int shmget(key_t key, size_t size, int shmflg);
```

在物理内存创建一个共享内存，返回共享内存的编号。

shmget最终创建的共享内存都会对应到同一块物理内存空间上。shm保存在物理内存，这样读写的速度要比磁盘要快，但是存储量不是特别大。


### 套接字

socket传送消息；