---
layout: post
title: Spark 计算框架：RDD 行动算子
categories: 大数据之kafka 大数据之spark
tags: scala java 大数据 kafka spark MacOS 环境搭建 Scala Maven Hadoop SQL 算子 数据分析 groupBy filter distinct coalesce shuffle 数据倾斜 分区 分组 聚合 关系型数据库 行动算子 转换算子 Driver Executor 闭包 序列化 
---

上一篇中讲到的转换算子都是得到新的RDD，而行动算子触发作业的执行！

collect() 就是一个典型的行动算，collect 算子会将不同分区的数据按照分区顺序采集到Driver 端内存中，形成数组。先看一下collect() 的代码实现

```scala
/**
 * Return an array that contains all of the elements in this RDD.
 *
 * @note This method should only be used if the resulting array is expected to be small, as
 * all the data is loaded into the driver's memory.
 */
def collect(): Array[T] = withScope {
  val results = sc.runJob(this, (iter: Iterator[T]) => iter.toArray)
  Array.concat(results: _*)
}


/**
 * Run a job on all partitions in an RDD and return the results in an array.
 *
 * @param rdd target RDD to run tasks on
 * @param func a function to run on each partition of the RDD
 * @return in-memory collection with a result of the job (each collection element will contain
 * a result from one partition)
 */
def runJob[T, U: ClassTag](rdd: RDD[T], func: Iterator[T] => U): Array[U] = {
  runJob(rdd, func, 0 until rdd.partitions.length)
}


/**
 * Run a function on a given set of partitions in an RDD and return the results as an array.
 *
 * @param rdd target RDD to run tasks on
 * @param func a function to run on each partition of the RDD
 * @param partitions set of partitions to run on; some jobs may not want to compute on all
 * partitions of the target RDD, e.g. for operations like `first()`
 * @return in-memory collection with a result of the job (each collection element will contain
 * a result from one partition)
 */
def runJob[T, U: ClassTag](
    rdd: RDD[T],
    func: Iterator[T] => U,
    partitions: Seq[Int]): Array[U] = {
  val cleanedFunc = clean(func)
  runJob(rdd, (ctx: TaskContext, it: Iterator[T]) => cleanedFunc(it), partitions)
}


/**
 * Run a function on a given set of partitions in an RDD and return the results as an array.
 * The function that is run against each partition additionally takes `TaskContext` argument.
 *
 * @param rdd target RDD to run tasks on
 * @param func a function to run on each partition of the RDD
 * @param partitions set of partitions to run on; some jobs may not want to compute on all
 * partitions of the target RDD, e.g. for operations like `first()`
 * @return in-memory collection with a result of the job (each collection element will contain
 * a result from one partition)
 */
def runJob[T, U: ClassTag](
    rdd: RDD[T],
    func: (TaskContext, Iterator[T]) => U,
    partitions: Seq[Int]): Array[U] = {
  val results = new Array[U](partitions.size)
  runJob[T, U](rdd, func, partitions, (index, res) => results(index) = res)
  results
}


/**
 * Run a function on a given set of partitions in an RDD and pass the results to the given
 * handler function. This is the main entry point for all actions in Spark.
 *
 * @param rdd target RDD to run tasks on
 * @param func a function to run on each partition of the RDD
 * @param partitions set of partitions to run on; some jobs may not want to compute on all
 * partitions of the target RDD, e.g. for operations like `first()`
 * @param resultHandler callback to pass each result to
 */
def runJob[T, U: ClassTag](
    rdd: RDD[T],
    func: (TaskContext, Iterator[T]) => U,
    partitions: Seq[Int],
    resultHandler: (Int, U) => Unit): Unit = {
  if (stopped.get()) {
    throw new IllegalStateException("SparkContext has been shutdown")
  }
  val callSite = getCallSite
  val cleanedFunc = clean(func)
  logInfo("Starting job: " + callSite.shortForm)
  if (conf.getBoolean("spark.logLineage", false)) {
    logInfo("RDD's recursive dependencies:\n" + rdd.toDebugString)
  }

  // 最后触发有向无环图调度器的执行
  dagScheduler.runJob(rdd, cleanedFunc, partitions, callSite, resultHandler, localProperties.get)
  progressBar.foreach(_.finishAll())
  rdd.doCheckpoint()
}
```

最后触发**有向无环图**调度器的执行！！

## reduce 算子

聚合功能

https://www.bilibili.com/video/BV11A411L7CK?p=82

```scala

```



## count 算子

```scala

```

## first 算子

```scala

```

## take 算子

```scala

```

## takeOrdered 算子

```scala

```

## aggregate 算子

https://www.bilibili.com/video/BV11A411L7CK?p=84

```scala

```

和aggregateByKey 的区别？首先aggregate 是行动算子，而aggregateByKey 是转换算子。还有一个主要的区别

## flod 算子

```scala

```

## countByKey 算子

```scala

```


## countByValue 算子




## 实现wordCount 的各种方式

https://www.bilibili.com/video/BV11A411L7CK?p=85

```scala

```

## foreach 算子

```scala
val rdd = sc.makeRDD(List(1,2,3,4), 2)

// 先采集再循环
// collect 算子会将不同分区的数据按照分区顺序采集到Driver 端内存中，形成数组
// 这个println 是在Driver 端执行的操作
println("rdd.collect().foreach(println)");
rdd.collect().foreach(println)

// 没有顺序的概念，这个println 其实是在Executor 端执行的操作
// 分布式执行相关逻辑，因为是分布式执行所以是没有办法控制顺序！
println("rdd.foreach(println)")
rdd.foreach(println)
```

为什么称为算子？因为RDD 的方法（算子）和Scala 集合对象的方法不一样。集合对象的方法都是在同一个节点的内存中完成的。而RDD 的方法是可以将计算逻辑发送到Executor 分布式执行的！！

为了区分，所以称RDD 的方法为算子！

RDD 的方法外部的操作都是在Driver 端执行的，而方法内部的逻辑代码都是在Executor 端执行的

```scala
// 这个println 方法是RDD.foreach() 算子的外部操作，在Driver 执行
println("rdd.foreach(println)")

// 这个println 方法是RDD.foreach() 算子的内部操作，在Executor 执行
rdd.foreach(println)
```

https://www.bilibili.com/video/BV11A411L7CK?p=89

## 闭包与序列化

https://www.bilibili.com/video/BV11A411L7CK?p=89


