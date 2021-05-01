---
title: linux c++ webServer
date: 2021-03-02 13:44:28
categories: 
- C++
tags: [C++]
keywords: [C++]
---

webServer面试问题相关

<!---more--->

## Reactor和Proator


两个与事件分离器有关的模式是Reactor和Proactor。
* Reactor模式采用同步IO，而Proactor采用异步IO。
* 在Reactor中，事件分离器负责等待文件描述符或socket为读写操作准备就绪，然后将就绪事件传递给对应的处理器，最后由处理器负责完成实际的读写工作。
* 而在Proactor模式中，处理器--或者兼任处理器的事件分离器，只负责发起异步读写操作。IO操作本身由操作系统来完成。传递给操作系统的参数需要包括用户定义的数据缓冲区地址和数据大小，操作系统才能从中得到写出操作所需数据，或写入从socket读到的数据。事件分离器捕获IO操作完成事件，然后将事件传递给对应处理器。

比如，在windows上，处理器发起一个异步IO操作，再由事件分离器等待IOCompletion事件。典型的异步模式实现，都建立在操作系统支持异步API的基础之上，我们将这种实现称为“系统级”异步或“真”异步，因为应用程序完全依赖操作系统执行真正的IO工作。举个例子，将有助于理解Reactor与Proactor二者的差异，以读操作为例（类操作类似）。

* 在Reactor中实现读：
  - 注册读就绪事件和相应的事件处理器
  - 事件分离器等待事件- 事件到来，激活分离器，分离器调用事件对应的处理器。
  - 事件处理器完成实际的读操作，处理读到的数据，注册新的事件，然后返还控制权。

* 在Proactor中实现读：
  - 处理器发起异步读操作（注意：操作系统必须支持异步IO）。在这种情况下，处理器无视IO就绪事件，它关注的是完成事件。
  - 事件分离器等待操作完成事件- 在分离器等待过程中，操作系统利用并行的内核线程执行实际的读操作，并将结果数据存入用户自定义缓冲区，最后通知事件分离器读操作完成。- 事件分离器呼唤处理器。
  - 事件处理器处理用户自定义缓冲区中的数据，然后启动一个新的异步操作，并将控制权返回事件分离器。可以看出，两个模式的相同点，都是对某个IO事件的事件通知(即告诉某个模块，这个IO操作可以进行或已经完成)。
  
  - 在结构上，两者也有相同点：demultiplexor负责提交IO操作(异步)、查询设备是否可操作(同步)，然后当条件满足时，就回调handler；

* 不同点在于，异步情况下(Proactor)，当回调handler时，表示IO操作已经完成；同步情况下(Reactor)，回调handler时，表示IO设备可以进行某个操作(can read or can write)。

作者：wuxinliulei

链接：https://www.zhihu.com/question/26943938/answer/68773398

来源：知乎

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

---

## eventfd

linux 2.6提供了eventfd系统调用来实现进程间通信；

---

## timer定时器的实现

### timerManager

负责管理timer，通过它可以添加timer，删除timer；

#### timerfd

首先它要create一个timerfd，开启一个channel供EPOLL监听；每次我们设定的timerfd到时后，epoll会返回这个fd和channel。

通过timerChannel的ReadCallback()，这时我们获取所有过期的定时器(getExpired())，然后对每一个定时器绑定的回调函数进行调用(timer->run())；

timerManager只需创建一个timerfd就可以了，当我们添加timer的时候，我们将timer插入到set中，然后查看第一个过期的timer有没有变化，如果有变化，我们用最新插入的timer的过期时间来更新timerfd的值。

#### TimerList
用set或者map存储每个Timer，每个Timer通过一个timestamp和Timer来标识，通过timestamp进行排序管理；

### timer的实现

