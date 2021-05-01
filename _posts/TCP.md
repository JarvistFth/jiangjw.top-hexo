---
title: 计算机网络 - TCP
date: 2021-03-02 13:44:28
categories: 
- 计网
- TCP
tags: [计网,TCP]
keywords: [计网,TCP]
---

TCP八股文
<!---more--->
## TCP

面向连接、可靠、基于字节流的传输层通信协议。

### TCP头

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20210302151027.png)

## TCP 特征：

1. 面向连接的；通信前需要建立连接；
2. 点对点通信；
3. 可靠交付；无差错、不丢失、不重复、按序到达；
4. 全双工通信；允许同时互相发送数据，两边都有缓存接收数据；
5. 面向字节流；数据流为字节序列。

## 如何保证可靠传输

### 确认和超时重传

### 流量控制

### 拥塞控制

#### 慢启动

不要一开始就发送大量的数据，先探测一下网络的拥塞程度，也就是说由小到大逐渐增加拥塞窗口的大小。

#### 拥塞避免

拥塞避免算法让拥塞窗口缓慢增长，即每经过一个往返时间RTT就把发送方的拥塞窗口cwnd加1，而不是加倍。这样拥塞窗口按线性规律缓慢增长。

#### 快重传

快重传要求接收方在收到一个失序的报文段后就立即发出重复确认（为的是使发送方及早知道有报文段没有到达对方）而不要等到自己发送数据时捎带确认。快重传算法规定，发送方只要一连收到三个重复ACK就应当立即重传对方尚未收到的报文段，而不必继续等待设置的重传计时器时间到期。

#### 快恢复

当发送方连续收到三个重复确认时，就执行“乘法减小”算法，把ssthresh门限减半。但是接下去执行的是拥塞避免算法。

### 数据校验

checkSum校验和。

## TCP建立的阶段

1. client SYN -> server；
2. client <- SYN,ACK server；
3. client ACK -> server. 

- 初始seq随机：为了防止伪造seq进行攻击。

### 为什么需要三次握手

1. 防止网络问题造成的旧重复连接重复建立；（通过seq判断）；
2. 可靠传输在不可靠信道上，双方都需要确认对方收到了自己发送的seq，至少需要三次通信；

## TCP释放阶段

1. client FIN -> server
2. client <- ACK server
3. client <- data/ FIN+ACK server
4. client ACK -> server

FIN-WAIT-1：等待server发送ack，客户端到server关闭连接；
FIN-WAIT-2：等待server将未发送完的数据发送完，同时server发送FIN，从server端关闭连接。

### 为什么是四次挥手

TCP是全双工通信，关闭连接要从客户端->server端 / server端->客户端。

TCP建立连接时，ACK和SYN一起发送，所以减少了一次；但是释放连接时，FIN和ACK是分开的。释放连接时，server受先要响应client的FIN，然后可能还有未发送完的数据；只有当发送完所以数据，才向client发送FIN。


### TIME-WAIT原因

1. 保证客户端发送的ACK能到达server端；若未成功到达，这时候server端会重传FIN+ACK，这时候客户端还要处理这个FIN+ACK，然后重传ACK给server，并重新进入TIME_WAIT；
2. 防止失效连接数据出现在本次连接中（在2MSL后，所有数据都会丢弃），下次再使用这个连接时，不会再收到旧的数据。

### 和UDP区别
1. TCP面向连接的，UDP无连接；
2. TCP保证可靠交付，UDP尽最大努力交付；
3. TCP面向字节流，UDP面向报文；
4. TCP包有最大字节数，UDP交IP分包；
5. TCP一对一，UDP可以多对一、一对多；
5. UDP适用于实时场景，TCP适合可靠传输场景。


## HTTP：

### Content-length的使用：
使用了Transfer-Encoding字段时，不能使用Content-length；比如http请求中的chunk模式；

#### chunk模式
在持久连接中，可能会一边生产数据，一边发送数据；这时候我们不知道内容的长度，需要一边生产数据一边发送数据；这时候就不能使用Content-Length；

Transfer-Encoding:Chunk 的首部设定被启用；

每个内容由若干个chunk组成，chunk编码格式如下：

[chunk size][\r\n][chunk data][\r\n][chunk size][\r\n][chunk data][\r\n][chunk size = 0][\r\n][\r\n]

chunk size是以十六进制的ASCII码表示，比如：头部是3134这两个字节，表示的是1和4这两个ascii字符，被http协议解释为十六进制数14，也就是十进制的20，后面紧跟[\r\n](0d 0a)，再接着是连续的20个字节的chunk正文。chunk数据以0长度的chunk块结束。