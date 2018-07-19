---
layout: post
title: 美丽的动态规划
categories: 深入学习之算法 深入学习之数据结构 重学算法与数据结构
tags: 算法 数据结构 动态规划 子问题 状态 状态转移函数 斐波那契 DP 最短路径 有向无环图 DAG 图论 递归 循环 C++ C 自顶向下的记忆法 自底向上 
---

>the basic idea of dynamic programming is to take a problem, split into subproblems, solve those subproblems and reuse the solutions to your subproblems.

## 斐波那契数列

首先是直接根据其递归的定义给出最简单的递归实现方式

```c++
int fib(int n){
    if(n <= 0)
        return 0;
    if(1 == n || 2 == n)
        return 1;
    return fib(n-1) + fib(n-2);
}
```

参考[《递归和循环》](http://www.xumenger.com/recursion-vs-loop-20170830/)，可以知道因为存在大量的重复运算，所以这种实现的复杂度达到`O(2^N)`

**自顶向下的记忆法**

每当我们计算出一个斐波那契数字，就把它放到一个字典中，然后当我们要计算某个斐波那契数字的时候，先检查字典中是否已经存在这个数，如果存在就直接拿出来，否则再进行计算

```c++
int compute(int n, int *memo){
    if(0 != memo[n])
        return memo[n];

    if(n <= 2)
        memo[n] = 1;
    else
        memo[n] = compute(n-1, memo) + compute(n-2, memo);

    return memo[n];
}

int fib(int n){
    if(n <= 0)
       return 0;
    int memo[n+1];
    memset(memo, 0, (n+1)*sizeof(int));
    return compute(n, memo);
}
```

用空间换时间的策略，把已经计算出来的结果保存起来，下次直接从对应的内存中获取，而不需要进行重复运算！现在的时间复杂度从 O(2^N) 降到 O(N)

>这正是动态规划的核心：先计算出子问题，再由子问题计父问题！但是在设计动态规划算法中真正的挑战在于如何正确找出什么是所谓的“子问题”！

**自底向上的动态规划**

上面的方法还是会用到递归，而且会有额外的空间消耗，很明显当n 很大的时候，会存在栈溢出的风险，其实还有更好的方法

```c++
int fib(int n){
    if(n <= 0)
        return 0;
    if(1 == n || 2 == n)
        return 1;
    int f1 = 1;
    int f2 = 1;
    int f = f1 + f2;
    int i = 3;
    while(i < n){
        f1 = f2;
        f2 = f;
        f = f1 + f2;
        i++;
    }
    return f;
}
```

这里解释一下所谓的自底向上和自顶向下！上面的自顶向下是要计算fib(n)，就要先计算出fib(n-1) 和fib(n-2)，然后逐步往下递归计算；而自底向上则是从fib(1)、fib(2) 往上递推得到fib(n)

当然！这只是一个极其简单的例子！！！！

**有向无环图(DAG)**

其实上面的斐波那契问题可以转换为有向无环图（DAG）的最短路径问题

仔细分析斐波那契数列的定义，其实fib(n) 的值只是依赖于fib(n-1) 和fib(n-2) 的值，像上面的**自底向上的动态规划**就是只需要保存倒数第一和倒数第二个值，然后就可以保证求出fib(n) 的值，这就像DAG 中的依赖关系

![image](../media/image/2018-07-19/01.png)

## 最短路径问题

比如下图怎么求解S 到V 的最短路径？

![image](../media/image/2018-07-19/02.png)

很明显，其**状态转移函数**应该为：min(S, T) = min{ min(A1, T) + path(S, A1),  min(B1, T) + path(S, B1), min(C1, T) + path(S, C1)}，然后依次类推求出 min(A1, T)、min(B1, T)、min(C1, T)，……

当然，根据上图，我们知道约束条件：min(A3, T) == path(A3, T)，min(B4, T) == path(B4, T)，min(C2, T) == path(C2, T)

比如这样一个求解有向无环图的最短路径问题。对应的Python 代码为

```python
'''
* W：权重图
* d[u]：u 到终点的距离
* s、t：起点和终点
'''
def dag_dp(W, s, t, d):
    if s == t:
        return 0;
    # 如果d[s]没有计算过，那么重新计算
    # 否则直接返回结果即可
    if s not in d:
        d[s] = min(W[s][v] + dag_dp(W, v, t, d) for v in W[s])
    return d[s]


DAG = {
    'a': {'b': 0},
    'b': {'c': 4, 'd': 6},
    'c': {'g': 2, 'h': -6},
    'd': {'f': 3, 'e': 5},
    'e': {'g': 0, 'h': -6},
    'f': {'i': -1},
    'g': {'h': 4},
    'h': {'i': 7},
    'i': {}
}
d = {}
print(dag_dp(DAG, 'a', 'i', d))
print(d)
```

运行结果为

![image](../media/image/2018-07-19/03.png)

很明显这是一个**自顶向下的记忆法**，使用d 这个字典来存储已经计算出来的结果！

## 参考资料

* [《递归和循环》](http://www.xumenger.com/recursion-vs-loop-20170830/)
* [MIT Algorithms](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/)
* [DP I](https://www.youtube.com/watch?v=OQ5jsbhAv_M)
* [DP II](https://www.youtube.com/watch?v=ENyox7kNKeY)
* [DP III](https://www.youtube.com/watch?v=ocZMDMZwhCY)
