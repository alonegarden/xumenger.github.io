---
layout: post
title: 搭建Spring Boot源码环境
categories: 大型系统架构 java之web开发 java之web框架 
tags: Java Web Flask Python Servlet Eclipse Spring SpringBoot maven gradle 
---

>本文基于Spring Boot-2.0.3

首先使用maven（不是gradle） 构建Spring Boot 项目（为了更快的构建，跳过测试用例）

```shell
$ ./mvnw clean install -DskipTests -Pfast
```

然后等待依赖包下载、等待编译

![](../media/image/2018-12-01/01.png)

执行下面的命令

```shell
$ ./mvnw eclipse:eclipse
```

![](../media/image/2018-12-01/02.png)

然后将项目导入到Eclipse 中

![](../media/image/2018-12-01/03.png)

打开SampleSimpleApplication 运行成功

![](../media/image/2018-12-01/04.png)

可以直接在浏览器中访问

![](../media/image/2018-12-01/05.png)
