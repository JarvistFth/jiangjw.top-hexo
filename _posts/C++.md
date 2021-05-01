---
title: C++ 面试八股
date: 2021-04-19 17:44:28
categories: 
- C++
tags: [C++]
keywords: [C++]
---


## lambda，std::function 等函数对象

函数类：一个重载了()运算符的类，使用就像函数调用一样。

lambda表达式：转换成类，通过[]捕获的变量传入成员变量，()参数为重载operator()函数的参数。

std::function是一个模板类，抽象了其从参数和返回值。

std::bind通过传入参数构造成一个函数对象。

## share_ptr<>/ weak_ptr<> / unique_ptr<>

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20210421121041.png)

share_ptr和weak_ptr共同继承自一个计数器抽象类，里面包括了强引用计数Uses和弱引用计数Weaks，通过构造函数初始化为1；

### unique_ptr<>

独占资源，不允许复制拷贝，但支持move；

### share_ptr<>

引用计数，支持拷贝，拷贝的话引用计数加1；析构时强引用计数-1，若强引用计数为0，释放内存；存在循环应用无法回收；引入weak_ptr<>；

### weak_ptr<>

只能通过share_ptr或weak_ptr构造，和share_ptr共享同一个计数器对象；

weak_ptr对象依赖shared_ptr/weak_ptr对象生成，并自动调用计数器递增弱引用计数。当某个weak_ptr生命期结束时，会自动调用析构函数，析构函数中会通过计数器递减弱引用计数，在强引用计数为0且释放资源的前提下，若弱引用计数为0，析构函数就会调用计数器的_Delete_this()函数删除计数器对象。


