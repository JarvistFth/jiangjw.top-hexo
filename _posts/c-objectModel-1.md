---
title: C++对象模型（1）
date: 2020-09-08 23:33:05
categories: 
- C++
tags: [C++]
keywords: [C++,对象模型]
---
C++对象模型基本内存分布。
<!---more--->

## C++对象模型

### 简单对象模型

一系列的对象是一串连续的slot，每个slot指向其中的一个成员。
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200908234058.png)

### 表格驱动对象模型

对象的slot里面存放指向一个table，每个table是一系列的slot，每个slot指向具体的member和function。
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200908234146.png)

### C++实际对象模型

* non-static member放在对象内；
* static member、 static function、non-static function放在object外；
* virtual function使用表格对象模型；
    1. class内生成一堆指向该class的virtual function的指针，放在表格内，表格成为virtual table。
    2. object内生成指针指向virtual table，称为vptr；vptr的重置和设置都由构造函数、析构函数和复制构造函数完成。
    3. type_info object放在table的首位，用于runtime type identification，RTTI。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200908234349.png)

## C++对象继承的内存模型

### 单继承

1. 不重写虚函数的单继承：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200912154315.png)

父类虚函数放在虚函数表的前面，子类自己的虚函数添加到虚函数表后；


2. 重写父类虚函数的单继承：
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200912154504.png)

子类重写父类虚函数时，将父类对应的虚函数覆盖为子类的实现。其他与不重写的一致。

其余继承的或新增的成员变量，都按照顺序排列在vptr后。

### 多继承

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200912154840.png)

1. 按照声明顺序，排列父类的vptr和成员变量；
2. 每个父类都有自己的vptr，指向一个属于自己的虚函数表；
3. 子类自己新增的虚函数放在第一个父类的虚函数表后面；
4. 如果父类有两个同名的虚函数，子类重写时都会重写为子类的虚函数，将两个父类的虚函数都进行了覆盖。

* 注：如果在实际中需要对多继承的同名虚函数进行不同的重写，需要在父类与子类中引入两个中间类。两个中间类分别声明一个纯虚函数，然后对父类的同名虚函数进行重写，重写时调用其声明的纯虚函数。这样子类再需要进行调用时，继承自中间类，然后利用多态特性，分别调用两个中间类中的同名虚函数。由于同名虚函数的实现是调用两个不同的纯虚函数，所以这样就可以进行同名虚函数的不同重写。


### 虚继承
![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200912161827.png)

虚基类为了解决多个间接父类的重复继承问题，以不能使用上面简单的扩充并为每个虚基类提供一个虚函数指针（这样会导致重复继承的基类会有多个虚函数表）形式。

因此，在虚继承中，派生类和基类的数据，是完全间隔的，先存放派生类自己的虚函数表和数据，中间以0x分界，最后保存基类的虚函数和数据。如果派生类重载了父类的虚函数，那么则将派生类内存中基类虚函数表的相应函数替换。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200912163008.png)

### 菱形继承

共同虚基类进行虚继承，直接继承的父类按照多继承的顺序进行排列。虚基类排在后面，直接继承的父类在前面。
子类重写覆盖所有父类包括虚基类的虚函数。

![](https://jaroffertree.oss-cn-hongkong.aliyuncs.com/20200912164817.png)

