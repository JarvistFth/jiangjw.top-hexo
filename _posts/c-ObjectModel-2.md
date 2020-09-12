---
title: c++ObjectModel-2
date: 2020-09-12 17:44:28
categories: 
- c++
tags: [c++]
keywords: [c++,对象模型]
---
构造函数语义学

<!---more--->

# 编译合成构造函数
只有四种情况编译器会为没有声明构造函数的类合成一个构造函数。

1. 这个类里面包含另一个类的成员变量，并且另一个类是有默认构造函数的；例子：
```C++
class Foo{
    public:
    Foo();
    Foo(int);
}

class Bar{
    public:
    Foo foo;
}

void foo_bar{
    Bar bar;//Bar::foo必须在此初始化；foo是它的一个成员变量且有默认构造函数，因此可以合成。
}
```

如果类有默认构造函数，但他没有对类内另一个类的成员变量做初始化，那么会在该默认构造函数内添加一些代码，用于完成其他类成员变量的初始化。例如：

```c++


2. D:\QtProject\build-qmc-decoder-Qt5-Desktop_Qt_5_13_1_MinGW_64_bit-Debug\debug\moc_mainwindow.cpp:80: error: undefined reference to `MainWindow::on_tableWidget_cellActivated(int, int)'

