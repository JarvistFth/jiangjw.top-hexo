---
title: redis设计与实现读书笔记(8)--订阅与退订
date: 2020-09-15 15:23:13
categories: redis
tags: [redis]
keywords: [redis]
---
redis的发布与订阅功能。

<!---more--->

### 概述
redis的订阅支持：
1. 订阅频道（SUBSCRIBE)
2. 订阅模式（PSUBSCRIBE）

两种模式。

### 频道的订阅与退订
当client执行SUBSCRIBE命令时，客户端与频道建立了订阅关系。

REDIS将所有频道的订阅关系都保存在redisServer中的pubsub_channels字典里面，字典的key是某个被订阅的频道名称，value是一个链表，链表里面记录了所有订阅这个频道的client。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200915161502.png)

#### 订阅频道
1. 如果频道还没有其他订阅者，那么该频道的key肯定不存在，所以在字典里面创建一个该频道的key，然后将客户端添加到value的链表中；

2. 如果频道已经有其他订阅者，就把客户端添加到value的链表后。

#### 退订频道
1. 在字典中的对应key的value的链表中删除该client；
2. 如果链表删除后为空，就从字典中删除该key。


### 模式的订阅与退订
REDIS将所有模式的订阅都保存在redisServer中的pubsub_patterns链表中；链表的每一项都保存了pubsubPatterns结构，pattern属性记录了被订阅的模式，client记录了订阅模式的客户端。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200915153751.png)

#### 订阅模式
使用PSUBSCRIBE命令来订阅模式。
1. 新建一个对应的pubsubPattern结构，将pattern属性设置为订阅的模式，将client设置为当前客户端；
2. 将pubsubPattern结构添加到上述pubsub_patterns链表的末尾。

#### 退订模式
PUNSUBSCRIBE，从链表中找到订阅了pattern属性的pubsubPattern结构然后将他们从链表中删除。

### 发送消息
PUBLISH 【CHANNEL】 【MESSAGE】 命令可以向该CHANNEL发送消息MESSAGE；这时候会：
1. 将message发送给channel频道的所有订阅者；
2. 如果有一个或多个模式pattern与频道channel相匹配，那么将消息message发送给pattern模式的订阅者。

#### 将消息发送给频道订阅者
通过pubsub_channel字典里面的key，找到key对应的链表，遍历链表里面的client，向client发送消息。

#### 将消息发送给模式订阅者
从pubsub_patterns链表中查找与被订阅模式匹配的结构，然后根据结构中的client属性，选择发送给client属性对应的客户端。

### 查看订阅信息
PUBSUB命令，可以查看某个频道有多少订阅者，或者某个模式有多少订阅者之类的。

这个命令也是通过遍历pubsub_patterns链表和pubsub_channel字典来获取对应信息的。


