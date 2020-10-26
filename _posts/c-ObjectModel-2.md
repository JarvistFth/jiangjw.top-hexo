---
title: C++对象模型（2）
date: 2020-09-12 17:44:28
categories: 
- C++
tags: [C++]
keywords: [C++,对象模型]
---
构造函数语义学

<!---more--->

## 编译合成构造函数
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

```C++
//原定义构造函数
Bar::Bar(){
    str = 0;
}

//扩张后构造函数
Bar::Bar(){
    foo.Foo::Foo();
    str = 0;
}
```


2. 如果一个没有任何构造函数的class派生自一个带有默认构造函数的base class，这个派生出来的class会合成默认构造函数。

3. class声明或继承一个virtual function；
4. class派生自一个继承串链，其中有一个或更多的virtual base classes。

其中3和4是为了让这个class的vptr有一个正确的初始化，指向编译产生的vtbl。

## 复制构造函数

复制构造函数如果没有被显式声明，就有可能被合成出来；

具体是否会合成，取决于该类是否展现出bitwise copy semantics（位逐次拷贝）。


###  展现位逐次拷贝的复制构造函数
递归方式进行成员变量初始化，从某一个object拷贝到另一个object上。

示例：

```C++
class String{
    public:
    // no explicit copy constructor
    private:
    char* str;
    int len;
}

class Word{
    public:
    // no explicit copy constructor
    private:
    int occurs;
    String words;
}

Word w("aa");
Word b = w;

//先对occurs进行初始化
//b.occurs = w.occurs
//然后对String类的words递归进行初始化
// b.words.str = w.words.str; b.words.len = w.words.len
```

### 不展现位逐次拷贝的情况，合成复制构造函数

1. 一个class内含有的member object声明有一个copy constructor；
2. 当class继承自一个base class，而base class有一个copy constructor；

上述1和2是为了能正确的调用copy constructor；

3. 当一个class声明了virtual function；
4. 当一个class派生自一个继承串链，其中有virtual base class的时候；

上述3和4是因为：位逐次拷贝不会对vptr进行处理。

3. 假如使用位逐次拷贝，将派生类拷贝给基类，那么基类object的vptr也会指向派生类。但是这时候根据运行时多态，实际上是调用基类的virtual function的时候，就会crash；所以我们要在合成的复制构造函数中调整vptr的指向；
4. 将派生类拷贝给基类，需要设定指向虚基类的vptr。


## 初始化列表
尽量使用初始化列表进行类的构造初始化，这样可以节省一些copy的操作，相当于原地赋值构造；

但是由于初始化列表可能会根据变量的声明顺序来重新进行排序安插代码，所以如果初始化的变量之间有依赖关系，建议最基础的变量放到初始化列表中，而其他有依赖关系的代码按照依赖关系在构造函数中调用。
