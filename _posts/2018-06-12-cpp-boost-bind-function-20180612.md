---
layout: post
title: boost::function和boost::bind
categories: c/c++之基础语法 c/c++之函数 c/c++之指针与内存 
tags: c++ boost bind function 函数指针 
---

## boost::function

boost::function是一个函数包装器，也即一个函数模板，可以用来代替拥有相同返回类型，相同参数类型，以及相同参数个数的各个不同的函数

```cpp
#include<boost/function.hpp>
#include<iostream>

using namespace std;

typedef boost::function<int(int ,char)> Func;

int test(int num,char sign)
{
   cout << num << sign << endl;
   return 0;
}

int main()
{
    Func f;
    f = &test;  //or f=test
    f(1,'A');
}
```

编译运行的效果如下

![](../media/image/2018-06-12/01.png)

当然有人会说，我用函数指针也可以做到啊

```cpp
#include<iostream>

using namespace std;

typedef int (*Func)(int, char);

int test(int num,char sign)
{
   cout << num << sign << endl;
   return 0;
}

int main()
{
    Func f;
    f = &test;  //or f=test
    f(1,'A');
}
```

当然也可以得到同样的效果

![](../media/image/2018-06-12/02.png)

但是为什么还要用boost::function呢？

>如果没有boost::bind，那么boost::function就什么都不是；而有了boost::bind，同一个类的不同对象可以delegate给不同的实现，从而实现不同的行为，简直就是无敌了

## boost::bind

boost::function就像C#中的delegate，可以指向任何函数，包括成员函数（这点就是普通的函数指针做不到的！）

当用bind把某个成员函数绑定到某个对象上的时候，就可以得到一个closure（闭包）

```cpp
#include <string>
#include <iostream>
#include<boost/function.hpp>
#include<boost/bind.hpp>

using namespace std;

class Foo{
    public:
        void methodA() { cout << "Foo::methodA()" << endl; }
        void methodInt(int a) { cout << "Foo::methodInt(" << a << ")" << endl; }
        void methodString(const string &str) { cout << "Foo::methodString(" << str << ")" << endl; }
};

class Bar{
    public:
        void methodB() { cout << "Foo::methodB()" << endl; }
};


int main()
{
    //无参数，无返回值
    boost::function<void()> fun;

    //调用foo.methodA()
    Foo foo;
    fun = boost::bind(&Foo::methodA, &foo);
    fun();

    //调用bar.methodB()
    Bar bar;
    fun = boost::bind(&Bar::methodB, &bar);
    fun();

    //调用foo.methodInt(42)
    fun = boost::bind(&Foo::methodInt, &foo, 42);
    fun();

    //调用foo.methodString("hello")
    fun = boost::bind(&Foo::methodString, &foo, "hello");
    fun();


    //int参数，无返回值
    boost::function<void(int)> fun2;
    fun2 = boost::bind(&Foo::methodInt, &foo, _1);
    fun2(100);
}
```

编译运行效果如下

![](../media/image/2018-06-12/03.png)

