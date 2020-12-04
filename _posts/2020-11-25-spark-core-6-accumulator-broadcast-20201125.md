---
layout: post
title: Spark 计算框架：Spark 累加器和广播变量
categories: 大数据之kafka 大数据之spark
tags: scala java 大数据 kafka spark MacOS 环境搭建 Scala Maven Hadoop SQL 算子 数据分析 groupBy filter distinct coalesce shuffle 数据倾斜 分区 分组 聚合 关系型数据库 行动算子 转换算子 Driver Executor 闭包 序列化 血缘 依赖 宽依赖 窄依赖 阶段 持久化 checkpoint 检查点 累加器 广播变量
---

Spark 计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景

* RDD：弹性分布式数据集
* 累加器：分布式共享只写变量
* 广播变量：分布式共享只读变量

比如我们想求RDD 所有数据的和，很简单，reduce 算子实现两两相加

```scala
val rdd = sc.makeRDD(List(1,2,3,4))

// reduce，包括分区内计算和分区间计算
val sum : Int = rdd.reduce(_ + _)

// 得到结果是10，没有问题
print(sum)
```

我们可不可以遍历RDD 的所有元素实现想相加，这不就是我们平时写代码常用的思维方式吗

```scala
val rdd = sc.makeRDD(List(1,2,3,4))

val sum = 0;
rdd.foreach(
    num => {
        sum += num
    }
)

// 但是得到结果是0，错误的！
print(sum)
```

对于后者，foreach() 是一个分布式的循环，foreach() 中的逻辑是分布式计算的，在local 模式下，是分布在多个线程中执行的，所以各个线程都有可能修改sum 的值，所以最终的sum 值是不确定的，但是就算是有并发安全问题，sum 也不应该是0 呀，为什么呢？

那就要看一下这个逻辑到底是怎么分布式执行的？！在分布式计算前，Driver 会把sum 的值发送给各个Executor，在各个Executor 中分布式执行，但是Executor 执行完了之后呢？按理应该是各个Executor 把结果返回给Driver，但是实际是没有返回结果的这个逻辑的，所以每次的增加都是Executor 中的sum 进行了变化，而Drvier 端的sum 是没有变化的

![](../media/image/2020-11-25-6/01.png)

## 累加器


## 广播变量


