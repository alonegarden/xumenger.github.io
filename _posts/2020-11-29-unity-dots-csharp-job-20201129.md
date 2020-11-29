---
layout: post
title: Unity DOTS 技术：C# Job
categories: 游戏开发之unity
tags: Unity UD ECS 性能 内存 DOD 面向数据编程 DOTS C# 委托 HybridECS Burst 
---

>Job System 是Unity 对CPU 多核编程的应用。通过把工作分散到CPU 的各个核心上来大大提升运行效率。而ECS 与它的搭配则是由于ECS 的System 部分天然是以批量处理为核心的，因此只要稍加改动，就可以转变为批量的分Job 交到多核去处理，实现性能的极大提升

>使用C# Job 开发，需要通过Package Manager 先安装这个包：Job

## C# Job 使用展示

首先独立于上文的案例，在场景中创建一个空物体，在物体上加一个MonoBehaviour

```c#

```

然后游戏运行起来的效果是这样的

![](../media/image/2020-11-29/01.png)

>这个是不是和[Java 线程池异步任务](http://www.xumenger.com/callable-future-20201105/) 的思想有点像？！

## 使用C# Job 优化程序

>Jobs -> Burst -> Enable Compilation 先不打开！另外，上面的案例也先从场景中删除！

在上文中，使用了HybridECS 开发了一个简单的案例，本文在上面代码的基础上添加C# Job，看一下优化后的效果

```c#

```

再次运行游戏，可以看一下FPS 等性能的指标

![](../media/2020-11-29/.gif)

现在再打开Profiler 看一下整体性能

![](../media/2020-11-29/.gif)

## 参考资料

* [UUG Online直播回放：DOTS从原理到应用-雨松MOMO](https://www.bilibili.com/video/BV1sD4y1Q7an)
* [DOTS深度研究之从原理到实践](https://www.xuanyusong.com/archives/4708)
* [Unity官方ECS样例](https://github.com/Unity-Technologies/EntityComponentSystemSamples.git)
* [Unity官方文档 ECS](https://docs.unity3d.com/Packages/com.unity.entities@0.16/manual/index.html)
* [Unity官方文档 Burst编译器](https://docs.unity3d.com/Packages/com.unity.burst@1.4/manual/index.html)
* [Unity官方文档 C# Job](https://docs.unity3d.com/Manual/JobSystem.html)
* [Unity官方文档 Mathematics](https://docs.unity3d.com/Packages/com.unity.mathematics@1.2/manual/index.html)
* [https://connect.unity.com/u/yu-song-momo-1](https://connect.unity.com/u/yu-song-momo-1)
* [Unity DOTS 介绍](https://www.bilibili.com/video/BV1tp4y1S7sc)
* [【游戏开发】Unity ECS DOTS 教程 （合集）机翻！](https://www.bilibili.com/video/BV1qE411x7Wg)
* [https://software.intel.com/sites/landingpage/IntrinsicsGuide/](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)
