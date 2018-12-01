---
layout: post
title: 搭建Spring Boot源码环境
categories: 大型系统架构 java之web开发 java之web框架 
tags: Java Web Flask Python Servlet Eclipse Spring SpringBoot maven gradle 
---

>本文基于[Spring Boot-2.0.3](https://github.com/spring-projects/spring-boot/releases/tag/v2.0.3.RELEASE)

将构建好的项目导入到Eclipse 中，选择【Import Existing Maven Projects】

![](../media/image/2018-12-01/01.png)

选择spring-boot-project、spring-boot-samples、spring-boot-tests 的所有项目（导入时间可能会比较久）

![](../media/image/2018-12-01/02.png)

>上面截图中我是一次性导入所有项目，结果堆内存不够用，所以建议一部分一部分的导入！

只需要导入spring-boot-project、spring-boot-samples 即可！

![](../media/image/2018-12-01/03.png)

选择spring-boot-sample-webflux 项目下的SampleWebFluxApplication.java，Run as Java Application，即可启动程序

![](../media/image/2018-12-01/04.png)

试一下，确实可以在浏览器中访问

![](../media/image/2018-12-01/05.png)

之前也整理过一篇[Spring Boot开发Web程序](http://www.xumenger.com/java-springboot-20180322/)，可以配合来理解Spring Boot！

## 参考资料

* [[源码分析]Spring boot 源码环境搭建](https://blog.csdn.net/u010536377/article/details/79517633)
* [Spring Boot开发Web程序](http://www.xumenger.com/java-springboot-20180322/)
* [搭建Flask源码环境](http://www.xumenger.com/pycharm-flask-20181202/)
* [搭建Hystrix开发环境](http://www.xumenger.com/hystrix-dev-20181125/)
* [Eclipse 错误；找不到或无法加载主类](https://blog.csdn.net/ljg888/article/details/7696488)
* [错误: 找不到或无法加载主类](https://www.cnblogs.com/wushuai2014/p/7468954.html)
