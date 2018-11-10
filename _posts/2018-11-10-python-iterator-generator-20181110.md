---
layout: post
title: Python迭代器和生成器
categories: python之基础 
tags: python python3 迭代器 生成器 yield generator iterator iter 可迭代对象 
---

## Python迭代器

我们已经知道，可以直接作用于for 循环的数据类型有以下几种

* 一类是集合数据类型，如list、tuple、dict、set、str 等
* 一类是generator，包括生成器和带yield 的generator function

这些可以直接作用于for 循环的对象统称为可迭代对象：Iterable，可以使用isinstance() 判断一个对象是否为Iterable 对象

```python
>>> from collections import Iterable
>>> isinstance([], Iterable)
True
>>> isinstance({}, Iterable)
True
>>> isinstance('abc', Iterable)
True
>>> isinstance((x for x in range(10)), Iterable)
True
>>> isinstance(100, Iterable)
False
```

而生成器不但可以用于for 循环，还可以被next() 函数不断调用并返回下一个值，直到最后抛出StopIteration 错误表示无法继续返回下一个值了

可以被next() 函数调用并不断返回下一个值的对象称为迭代器：Iterator，可以使用isinstance() 判断一个对象是否是Iterator 对象

```python
>>> from collections import Iterator
>>> isinstance((x for x in range(10)), Iterator)
True
>>> isinstance([], Iterator)
False
>>> isinstance({}, Iterator)
False
>>> isinstance('abc', Iterator)
False
```

生成器都是Iterator 对象，但list、dict、str 虽然是Iterable，但不是Iterator。把list、dict、str 等变成Iterator 可以用iter() 函数

```python
>>> isinstance(iter([]), Iterator)
True
>>> isinstance(iter('abc'), Iterator)
True
>>> it = iter('abc')
>>> next(it)
'a'
>>> next(it)
'b'
>>> next(it)
'c'
>>> next(it)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

你可能会问，为什么list、dict、str 等数据类型不是Iterator ？

这是因为Python 的Iterator 对象表示的是一个数据流，Iterator 对象可以被next() 函数调用并不断返回下一个数据，直到没有数据时抛出StopIteration 错误。可以把这个数据流看做一个有序序列，但我们却不能提前知道序列的长度，只能不断通过next() 函数实现按需计算下一个数据，所以Iterator 的计算是惰性的，只有在需要返回下一个数据时它才会计算

Iterator 甚至可以表示一个无限大的数据流，例如全体自然数，而使用list 是永远不可能存储全体自然数的，因为根本不可能有这么大的内存，而使用Iterator 只有在调用next() 时才按需计算下一个数据，所以不需要事先申请大内存！

## Python生成器

创建Python迭代器的功能虽然强大，但很多时候使用不方便。生成器是一个简单的方式来完成迭代。简单的说，Python的生成器是一个返回可迭代对象的函数

在一个一般函数中使用yield 关键字，可以实现一个最简单的生成器，此时这个函数变成一个生成器函数。yield 与return 返回相同的值，区别在于return 返回后，函数状态终止，而yield 会保存当前函数的执行状态，在返回后，函数又回到之前保存的状态继续执行

生成器函数与一般函数的不同

* 生成器函数包含一个或多个yield
* 当调用生成器函数时，函数将返回一个对象，但不会立即向下执行
* 像\_\_iter\_\_() 和\_\_next\_\_() 方法等都是自动实现的，所以我们可以通过next() 方法对对象进行迭代
* 一旦函数被yield，函数会暂停，控制权返回调用者
* 局部变量和它们的状态会被保存，直到下一次调用
* 函数终止的时候，StopIteration 会被自动抛出

```python
# 逆序yield出对象的元素
def rev_str(my_str):
    length = len(my_str)
    for i in range(length-1, -1, -1):
        yield my_str[i]

for char in rev_str("hello"):
    print(char)
```

生成器的表达式

```python
# Python中，有一个列表生成方法，比如
[x for x in range(5)]

# 如果[]换成()，那么会成为生成器的表达式
(x for x in range(5))

# 具体用法
a = (x for x in range(10))
b = [x for x in range(10)]
# 这是错误的，因为生成器不能直接给出长度
# print("length a: ", len(a))

# 输出列表长度
print("length b: ", len(b))

b = iter(b)
# 二者输出等价，不过b是在运行时开辟内存，而a是直接开辟内存
print(next(a))
print(next(b))
```

为什么使用生成器？

* 更容易使用，代码量较小
* 内存使用更加高效。比如列表是在建立的时候就分配所有的内存空间，而生成器仅仅是需要的时候才使用，更像是一个记录
* 代表了一个无限的流。如果我们要读取并使用的内容远远超过内存，但是需要对所有的流中的内容进行处理，那么生成器是一个很好的选择，比如可以让生成器返回当前的处理状态，由于它可以保存状态，那么下一次直接处理即可
* 流水线生成器

假设我们有一个快餐记录，这个记录的第4行记录了过去5年每小时售出的食物数量，并且我们要把所有的熟练加载一起，求解过去5年售出的总数。假设所有的数据都是字符串，并且不可用的数字被标记为N/A，那么可以这样处理

```python
with open("sells.log") as file:
    pizza_col = (line[3] for line in file)
    per_hour = (int(x) for x in pizza_col if x != 'N/A')   # 使用生成器进行自动迭代
    print("Total pizzas sold = ", sum(per_hour))
```

## Python迭代器、生成器进阶

通过列表生成式，我们可以直接创建一个列表。但是，受到内存限制，列表容量肯定是有限的。而且创建一个包含100 万个元素的列表，不仅占用很大的存储空间，如果我们仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了

所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python 中，这种一边循环一遍计算的机制，成为生成器：generator

要创建一个generator，有很多种方法。第一种方法很简单，只要把一个列表生成式的[]改成()，就创建了一个generator

```python
>>> L = [x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x10d327048>
```

generator 保存的是算法，每次调用next(g)，就计算出g 的下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出StopIteration 错误

当然，不断调用next(g) 实在变态，正确的方法是使用for 循环，因为generator 也是可迭代对象

```python
>>> next(g)
0
>>> next(g)
1
>>> for n in g:
...     print(n)
... 
4
9
16
25
36
49
64
81
```

所以，我们创建了一个generator 后，基本上永远不会调用next()，而是通过for 循环来迭代它，并且不需要关心StopIteration 的错误

generator 非常强大。如果推算的算法比较复杂，用类似列表生成式的for 循环无法实现的时候，还可以用函数来实现。比如，著名的斐波拉契数列（Fibonacci），除第一个和第二个数外，任意一个数都可由前两个数相加得到

>1, 1, 2, 3, 5, 8, 13, 21, 34, ...

斐波那契数列用列表生成式写不出来，但是用函数把它打印出来却很容易

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print(b)
        a, b = b, a + b
        '''
        等价于
        t = (b, a + b)
        a = t[0]
        b = t[1]
        '''
        n = n + 1
    return 'done'
```

上面的函数可以输出斐波那契数列的前个数

```python
>>> fib(5)
1
1
2
3
5
'done'
```

仔细观察，可以看出，fib 函数实际上是定义了斐波那契数列的推算规则，可以从第一个元素开始，推算出后续任意的元素，这种逻辑其实非常类似generator

也就是说，上面的函数和generator 仅一步之遥。要把fib函数变成generator ，只需要把print(b) 改为yield b 即可

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return "done"
```

这就是定义generator 的另一种方法。如果函数定义中包含了yield 关键字，那这个函数就不再是一个普通函数，而是一个generator

```python
>>> f = fib(5)
>>> for num in f:
...     print(num)
... 
1
1
2
3
5
```

比如创建一个生成自然数的生成器

```python
>>> def number():
...     begin = 0
...     while True:
...         yield begin
...         begin = begin + 1
... 
>>> n = number()
>>> next(n)
0
>>> next(n)
1
>>> next(n)
2
>>> 
```
