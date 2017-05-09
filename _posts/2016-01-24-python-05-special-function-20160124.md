---
layout: post
title: Python之特殊方法与多范式
categories: python之基础
tags: python 
---

转载自[Vamei：Python深入01 特殊方法与多范式](http://www.cnblogs.com/vamei/archive/2012/11/19/2772441.html)

Python一切皆对象，但同时，Python还是一个多范式语言(multi-paradigm)，你不仅可以使用面向对象的方式来编写程序，还可以用面向过程的方式来编写相同功能的程序(还有函数式、声明式等，我们暂不深入)。Python的多范式依赖于Python对象中的特殊方法(special method)。

特殊方法名的前后各有两个下划线。特殊方法又被成为魔法方法(magic method)，定义了许多Python语法和表达方式，正如我们在下面的例子中将要看到的。当对象中定义了特殊方法的时候，Python也会对它们有“特殊优待”。比如定义了`__inti__()`方法的类，会在创建对象的时候自动执行`__init__()`方法中的操作。

可以通过dir()来查看对象所拥有的特殊方法，比如dir(1)，输出

```
>>> dir(1)
['__abs__', '__add__', '__and__', '__class__', '__cmp__', '__coerce__', '__delattr__', '__div__', '__divmod__', '__doc__', '__float__', '__floordiv__', '__format__', '__getattribute__', '__getnewargs__', '__hash__', '__hex__', '__index__', '__init__', '__int__', '__invert__', '__long__', '__lshift__', '__mod__', '__mul__', '__neg__', '__new__', '__nonzero__', '__oct__', '__or__', '__pos__', '__pow__', '__radd__', '__rand__', '__rdiv__', '__rdivmod__', '__reduce__', '__reduce_ex__', '__repr__', '__rfloordiv__', '__rlshift__', '__rmod__', '__rmul__', '__ror__', '__rpow__', '__rrshift__', '__rshift__', '__rsub__', '__rtruediv__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__truediv__', '__trunc__', '__xor__', 'bit_length', 'conjugate', 'denominator', 'imag', 'numerator', 'real']
```

## 运算符

Python的运算符是通过调用对象的特殊方法实现的。比如：

```
'abc' + 'xyz'               # 连接字符串
```

实际执行了如下操作：

```
'abc'.__add__('xyz')
```

所以，在Python中，两个对象是否能进行加法运算，首先就要看相应的对象是否有`__add__()`方法。一旦相应的对象有`__add__()`方法，即使这个对象从数学上不可加，我们都可以用加法的形式，来表达`obj.__add__()`所定义的操作。在Python中，运算符起到简化书写的功能，但它依靠特殊方法实现。

Python不强制用户使用面向对象的编程方法。用户可以选择自己喜欢的使用方式(比如选择使用+符号，还是使用更加面向对象的`__add__()`方法)。特殊方法写起来总是要更费事一点。

尝试下面的操作，看看效果，再想想它的对应运算符

```
(1.8).__mul__(2.0)
True.__or__(False)
```

## 内置函数

与运算符类似，许多内置函数也都是调用对象的特殊方法。比如：

```
len([1,2,3])        # 返回表中元素的总数
```

实际上做的是：

```
[1,2,3].__len__()
```

相对与`__len__()`，内置函数len()也起到了简化书写的作用。

尝试下面的操作，想一下它的对应内置函数

```
(-1).__abs__()
(2.3).__int__()
```

## 表(list)元素引用

下面是我们常见的表元素引用方式

```
li = [1,2,3,4,5,6]
print(li[3])
```

上面的程序运行到li[3]的时候，Python发现并理解[]符号，然后调用`__getitem__()`方法。

```
li = [1, 2, 3, 4, 5, 6]
print(li.__getitem__(3))
```

尝试看下面的操作，想想它的对应

```
li.__setitem__(3, 0)
{'a':1, 'b':2}.__delitem__('a')
```

## 函数

我们已经说过，在Python中，函数也是一种对象。实际上，任何一个有`__call__()`特殊方法的对象都被当作是函数。比如下面的例子:

```
class SampleMore(object):
    def __call__(self, a):
        return a + 5
        
add = SampleMore()     # A function object
print(add(2))          # Call function    
map(add, [2, 4, 5])    # Pass around function object
```

add为SampleMore类的一个对象，当被调用时，add执行加5的操作。add还可以作为函数对象，被传递给map()函数。

当然，我们还可以使用更“优美”的方式，想想是什么。

## 总结

对于内置的对象来说(比如整数、表、字符串等)，它们所需要的特殊方法都已经在Python中准备好了。而用户自己定义的对象也可以通过增加特殊方法，来实现自定义的语法。特殊方法比较靠近Python的底层，许多Python功能的实现都要依赖于特殊方法。我们将在以后看到更多的例子。
