---
layout: post
title: Unity Dots 技术：C# Job
categories: 游戏开发之unity
tags: Unity UD ECS 性能 内存 DOD 面向数据编程 Dots C# 委托 HybridECS Burst 
---

>Job System 是Unity 对CPU 多核编程的应用。通过把工作分散到CPU 的各个核心上来大大提升运行效率。而ECS 与它的搭配则是由于ECS 的System 部分天然是以批量处理为核心的，因此只要稍加改动，就可以转变为批量的分Job 交到多核去处理，实现性能的极大提升



## 参考资料

视频相关

* [Unity DOTS技术详解 - 宣雨松](https://www.bilibili.com/video/BV18J411t7G8)
* [UUG Online直播回放：DOTS从原理到应用-雨松MOMO](https://www.bilibili.com/video/BV1sD4y1Q7an)
* [Unity DOTS 介绍](https://www.bilibili.com/video/BV1tp4y1S7sc)
* [[天天直播] 使用C# Job System并行化Dynamic Bone](https://www.bilibili.com/video/BV1Q741177Jd)
* [DOTS深度研究之从原理到实践](https://www.xuanyusong.com/archives/4708)

文档资料

* [DOTS原理与ECS实战：以ECS搜寻目标为例](https://www.bilibili.com/video/BV1xK4y1v7rw)
* [Unity官方ECS样例](https://github.com/Unity-Technologies/EntityComponentSystemSamples.git)
* [Unity官方文档 ECS](https://docs.unity3d.com/Packages/com.unity.entities@0.16/manual/index.html)
* [Unity官方文档 Burst编译器](https://docs.unity3d.com/Packages/com.unity.burst@1.4/manual/index.html)
* [Unity官方文档 C# Job](https://docs.unity3d.com/Manual/JobSystem.html)
* [Unity官方文档 Mathematics](https://docs.unity3d.com/Packages/com.unity.mathematics@1.2/manual/index.html)

雨松MoMo

* [https://connect.unity.com/u/yu-song-momo-1](https://connect.unity.com/u/yu-song-momo-1)
* [UUG Online直播回放：DOTS从原理到应用-雨松MOMO](https://www.bilibili.com/video/BV1sD4y1Q7an)