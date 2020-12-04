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

https://www.bilibili.com/video/BV11A411L7CK?p=105