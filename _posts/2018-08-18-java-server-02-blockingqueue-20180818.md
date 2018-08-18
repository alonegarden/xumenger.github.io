---
layout: post
title: Java服务端开发：阻塞队列
categories: 大型系统架构 java之多线程 java之面向对象
tags: Java服务端开发 Java 服务端 服务器 阻塞队列 BlockingQueue 阻塞 非阻塞 poll 生产者 消费者 多线程 线程安全 数据机构 
---

阻塞队列是Java 的util.concurrent 包下重要的数据结构，BlockingQueue 提供了线程安全的队列访问方式：当往阻塞队列中插入数据，且队列已满时，线程将会阻塞等待队列非满；当从阻塞队列取数据，且队列为空时，线程将会阻塞等待队列非空

JDK7 提供了以下7 个阻塞队列

* ArrayBlockingQueue：由数组结构组成的有界阻塞队列
* LinkedBlockingQueue：由链表结构组成的有界阻塞队列
* PriorityBlockingQueue：支持优先级排序的无阻塞队列
* DelayQueue：使用优先级队列实现的无界阻塞队列
* SynchronousQueue：不存储元素的阻塞队列
* LinkedTransferQueue：链表结构组成的无界阻塞队列
* LinkedBlockingDeque：链表结构组成的双向阻塞队列

阻塞队列提供了下列四种处理方法

 操作 | 抛异常      | 特定值    | 阻塞     | 超时 
-----|------------|----------|----------|-----------------------------
插入  | add(o)     | offer(o) | put(o)  | offer(o, timeout, timeunit)
移除  | remove(o)  | poll(o)  | take(o) | poll(timeout, timeunit)
检查  | element(o) | peek(o)  |         |

## 