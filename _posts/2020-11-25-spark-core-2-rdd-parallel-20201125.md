---
layout: post
title: Spark 计算框架：Spark Core RDD 并行计算
categories: 大数据之kafka 大数据之spark
tags: scala java 大数据 kafka spark MacOS 环境搭建 Scala Maven Hadoop RDD 累加器 广播变量 装饰者 设计模式 IO 分区 并行度 算子 转换算子 行动算子 
---

直接看一个基于map() 算子的例子

```scala
package com.xum.rdd

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext

object TestRDDParallel_1 {
  def main(args: Array[String]): Unit = {
    // 创建Spark 运行配置对象，连接。注意这里为了测试并行，要用[*]
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)
    
    val rdd = sc.makeRDD(List(1,2,3,4))
    
    // 通过转换算子，得到第一个新的RDD
    val rdd1 = rdd.map(
      num => {
        println("rdd1 ===> " + num)
        num
      }
    )
    
    // 通过转换算子，得到第二个新的RDD
    val rdd2 = rdd1.map(
      num => {
        println("rdd2 ---> " + num)
        num
      }
    )
    
    // 通过rdd2 的collect() 触发运算执行
    rdd2.collect()
    
    sc.stop()
  }
}
```

运行效果如下，可以看到计算顺序是“乱掉的”！

![](../media/image/2020-11-25/01-01.png)

>根据输出中的红色内容，可以看到，是有8 个Executor 的，因为上面设置了local[*]，所以默认当前本机的核数是多少，比如8核，那么就会用8个线程模拟运行场景

## 并发度为1的执行情况

在makeRDD() 时，指定一个分区，以控制并发度为1，然后看一下运行效果

```scala
package com.xum.rdd

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext

object TestRDDParallel_1 {
  def main(args: Array[String]): Unit = {
    // 创建Spark 运行配置对象，连接
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)
    
    // 强制设置只有一个分区，已实现控制并发度为1
    val rdd = sc.makeRDD(List(1,2,3,4), 1)
    
    // 通过转换算子，得到第一个新的RDD
    val rdd1 = rdd.map(
      num => {
        println("rdd1 ===> " + num)
        num
      }
    )
    
    // 通过转换算子，得到第二个新的RDD
    val rdd2 = rdd1.map(
      num => {
        println("rdd2 ---> " + num)
        num
      }
    )
    
    // 通过rdd2 的collect() 触发运算执行
    rdd2.collect()
    
    sc.stop()
  }
}
```

按照List(1,2,3,4) 中的元素，逐个先执行第一个转换算子，再执行第二个转换算子。所以对于RDD，一个分区内的数据是顺序一个一个执行逻辑的！只有前面一个数据的全部逻辑执行完毕后，才会执行下一个数据！

![](../media/image/2020-11-25/01-02.png)

>同样可以通过红色的输出看到，这种情况下只有一个Executor！

## 继续设置2个分区呢

```scala
package com.xum.rdd

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext

object TestRDDParallel_1 {
  def main(args: Array[String]): Unit = {
    // 创建Spark 运行配置对象，连接
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)
    
    // 强制设置只有一个分区，已实现控制并发度为1
    val rdd = sc.makeRDD(List(1,2,3,4), 2)
    
    // 通过转换算子，得到第一个新的RDD
    val rdd1 = rdd.map(
      num => {
        println("rdd1 ===> " + num)
        num
      }
    )
    
    // 通过转换算子，得到第二个新的RDD
    val rdd2 = rdd1.map(
      num => {
        println("rdd2 ---> " + num)
        num
      }
    )
    
    // 通过rdd2 的collect() 触发运算执行
    rdd2.collect()
    
    sc.stop()
  }
}
```

解释一下下面的执行效果，首先红色输出显示，当前是2 个Executor 执行的！另外可以看到是1、3先执行完的，然后2、4 执行，因为按照Spark 的分区规则，1、2 是分到一个分区中的，3、4 是分到另一个分区的，1、2 只能顺序执行，3、4 只能顺序执行，但是1、3 在两个分区，可以并行！

![](../media/image/2020-11-25/01-03.png)
