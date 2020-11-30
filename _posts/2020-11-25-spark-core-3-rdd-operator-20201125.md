---
layout: post
title: Spark 计算框架：RDD 常用算子原理
categories: 大数据之kafka 大数据之spark
tags: scala java 大数据 kafka spark MacOS 环境搭建 Scala Maven Hadoop SQL 算子 数据分析 groupBy filter distinct coalesce shuffle 数据倾斜 
---


## groupBy算子

Spark分组与分区

groupBy会将数据打乱（打散），重新组合，这个操作称为shuffle！

使用groupBy统计apache.log，分析某个时间段的点击量

https://www.bilibili.com/video/BV11A411L7CK?p=52


## filter算子

计算规则是对每个分区做计算的

按照一定条件过滤，可能存在分区1剩下1条，分区2剩下10000条，这个称为数据倾斜

使用filter从服务器日志数据apache.log 中获取2015年5月17日的请求路径

```scala
val rdd = sc.textFile("datas/apache.log")
rdd.filter(
    line => {
        val datas = line.split(" ")
        val time = datas(3)
        time.startsWith("17/05/2015")
    }
).collect().foreach(println)
```

可以用sample算子来协助解决数据倾斜


## distinct算子

Scala中集合的distinct去重是使用HashSet实现的

而在Spark 中，数据分布在不同的分区中，如何实现去重？

```scala
// TODO 补充distinct 代码

map(x => (x, null)).reduceByKey((x, _) => x, numPartitions).map(_._1)
```

分布式的处理方式来实现去重


## coalesce算子

根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率

当Spark 程序中，存在过多的小任务时，可以通过coalesce方法，收缩合并分区，减少分区的个数，减小任务调度成本

但是这个算子缩减分区可能导致数据不均衡，如果想要数据均衡，则进行shuffle 处理，即该算子的第二个参数设置为true

https://www.bilibili.com/video/BV11A411L7CK?p=58

比如可能发现分区太少了，每个分区的数据量太大，就想要扩大分区，使每个分区中的数据量稍微少一些。分区数变多后，也可以增强并行计算的能力

coalesce 算子也可以扩大分区，此时shuffle 参数必须指定为true！要打乱数据然后重新组合！

扩大分区也有一个专门的算子：repartition，其底层就是调用coalesce


## sortBy算子

```scala
val rdd = sc.makeRDD(List(("1", 1), ("2", 2), ("22", 3), (4", 4)))

// 按照键值降序排序
val newRDD = rdd.sortBy(t => t._1, false)

newRDD.saveAsTextFile("output")
```

排序前后，分区数不变，但因为做了排序，数据可能从原来的分区放到其他的分区，底层会有shuffle 操作


## 双值算子

```scala
val rdd1 = sc.makeRDD(List(1, 2, 3, 4))
val rdd2 = sc.makeRDD(List(3, 4, 5, 6))

// 交集
val rdd3 = rdd1.intersection(rdd2)
println(rdd3.collect().mkString(","))

// 并集
val rdd4 = rdd1.union(rdd2)
println(rdd4.collect().mkString(","))

// 差集
val rdd5 = rdd1.subtract(rdd2)
println(rdd5.collect().mkString(","))

// 前面三个如果两个RDD 的数据类型不一致，编译报错！
// 拉链算子，可以处理两个数据类型不一致的RDD，但是要求两个RDD 的分区数一致！每一个分区的数据量也要一致！
val rdd6 = rdd1.zip(rdd2)
println(rdd6.collect().mkString(","))
// (1,3),(2,4),(3,5),(4,6)
```


## partitionBy算子处理键值对

```scala
val rdd = sc.makeRDD(List(1, 2, 3, 4))

// 转换成tuple 类型
val mapRDD : RDD[(Int, Int)] = rdd.map((_, 1))

// 隐式转换：RDD => PairRDDFunctions
// 根据指定的分区规则，对数据进行重分区
val newRDD = rdd.partitionBy(new HashPartitioner(2))
```

partitionBy() 是PairRDDFunctions 中的一个方法，不是RDD 的算子！

因为RDD 中有一个方法（待补充）

```scala
implicit def rddToPairRDDFunctions[K, V]
....
```

>扩展内容：分区器！当然也可以自己实现一个分区器！


## reduceByKey算子

```scala
val rdd = sc.makeRDD(List(("a", 1), ("a", 2), ("b", 4)))

// reduceByKey 相同key 的数据进行value 的聚合
val reduceDD = rdd.reduceByKey((x:Int, y:Int) => {x + y})
```