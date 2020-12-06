---
layout: post
title: Unity 可编程渲染管线
categories: 游戏开发之unity
tags: Unity UD 渲染管线 HDRP GPU 可编程渲染管线 高清渲染管线 Unity3D URP HDRP Shader 剔除 渲染 后处理 DrawCall 内置管线 
---

可编程渲染管线（Scriptable Render Pipeline），支持直接通过C# 脚本控制渲染管线。目前Unity 官方提供高清渲染管线（HDRP）和轻量级渲染管线（LWRP），前者专注于高端图形渲染，为支持Compute Shader 的现代平台优化，可配置的集合了Tile/Cluster Deferred/Forward 光照；后者专注于性能，支持所有平台，Single-Pass 前向渲染

![](../media/image/2020-12-07/01.png)

渲染管线最主要需要关注的事情包括：剔除（Culling）、渲染（Render）、后期处理（Post Processing）

## URP/LWRP

Unity 已经正式将LWRP 的名称变更为Universal RP，即通用渲染管线，并将正式接过内置管线的大旗，成为新一代Unity 的兼容所有平台的通用渲染管线

URP 是单Pass 前向渲染管线，而内置管线是多Pass 前向渲染管线和延迟渲染管线。URP 没有延迟渲染，因此我们只对比前向渲染这一项（其实手游也基本只会用前向渲染，延迟渲染的G-Buffer 所需要的带宽带来的开销太大）

所谓的前向渲染，就是在渲染物体受光点光照的时候，分别对每个点光对该物体产生的影响进行计算，最后将所有光的渲染结果相加得到最终物体的颜色。内置管线的做法是，用多个Pass 来渲染光照，第一个Pass 只渲染主光源，然后多出来的光每个光用一个Pass 单独渲染，这也是为什么在做手游的时候很少会用点光源，因为对于内置管线来说，每多一盏光，整个场景的DrawCall 就会翻倍，这个性能开销是无法接受的

URP 的做法则是在一个Pass 当中，对这个物体收到的所有光源通过一个for 循环一次性计算，这样一个物体的光照可以在一次DrawCall 中计算完毕，省去多个Pass 的上下文切换以及光栅化等开销。不过URP 只支持一盏直光，单个物体最多支持4 盏点光，单个相机最多支持16 盏灯光

用了URP 之后，只要控制好点光的范围，在手游里面也可以做多点光照明了，例如释放一个火球照亮周围物件，这在内置管线里基本是可以放弃的功能（或者用其他作假的方式模拟）

比如从github 下载一个URP 的Unity 项目，在本机打开后可能有这样的错误

![](../media/image/2020-12-07/02.png)

```
Library/PackageCache/com.unity.render-pipelines.universal@7.5.1/Runtime/ForwardRenderer.cs(484,31): 
error CS1061: 'ScriptableCullingParameters' does not contain a definition for 'maximumVisibleLights' 
and no accessible extension method 'maximumVisibleLights' accepting a first argument of type 
'ScriptableCullingParameters' could be found (are you missing a using directive or an assembly reference?)
```

需要为当前项目设置URP（通用渲染管线）

Projects窗口 -> Assets -> Create -> Rendering -> Universal Render Pipeline -> Pipeline Asset (Forward Renderer)

菜单 -> Edit -> Project Settings -> Graphics 在Scriptable Redner Pipeline Settings 选择刚才创建的URP 资源

![](../media/image/2020-12-07/03.png)

## HDRP



## 参考资料

* [[官方直播] Unity HDRP高清渲染管线 – 艺术家工作流](https://www.bilibili.com/video/BV1Bt41127ji)
* [走进LWRP（Universal RP）的世界](https://connect.unity.com/p/zou-jin-lwrp-universal-rp-de-shi-jie)