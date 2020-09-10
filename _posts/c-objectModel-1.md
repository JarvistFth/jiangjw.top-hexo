---
title: c++-objectModel-1
date: 2020-09-08 23:33:05
categories: 
- c++
tags: [c++]
keywords: [c++,对象模型]
---

# C++对象模型

## 简单对象模型

一系列的对象是一串连续的slot，每个slot指向其中的一个成员。
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200908234058.png)

## 表格驱动对象模型

对象的slot里面存放指向一个table，每个table是一系列的slot，每个slot指向具体的member和function。
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200908234146.png)

## C++实际对象模型

* non-static member放在对象内；
* static member、 static function、non-static function放在object外；
* virtual function使用表格对象模型；
    1. class内生成一堆指向该class的virtual function的指针，放在表格内，表格成为virtual table。
    2. object内生成指针指向virtual table，称为vptr；vptr的重置和设置都由构造函数、析构函数和复制构造函数完成。
    3. type_info object放在table的首位，用于runtime type identification，RTTI。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200908234349.png)

# C++对象继承的内存模型

## 单继承

## 多继承

## 虚继承

## 菱形继承

## 
