---
layout: post
title: Spark 计算框架：RDD 行动算子
categories: 大数据之kafka 大数据之spark
tags: scala java 大数据 kafka spark MacOS 环境搭建 Scala Maven Hadoop SQL 算子 数据分析 groupBy filter distinct coalesce shuffle 数据倾斜 分区 分组 聚合 关系型数据库 行动算子 转换算子 
---

上一篇中讲到的转换算子都是得到新的RDD，而行动算子触发作业的执行！collect() 就是一个典型的行动算子，先看一下collect() 的代码实现

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

最后触发有向无环图调度器的执行！！

## reduce 算子

聚合功能



## 