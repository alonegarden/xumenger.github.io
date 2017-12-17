---
layout: post
title: 时序数据库influxdb
categories: 数据库之influxdb python之数据库编程
tags: 数据库 时序数据库 influxdb python python3 docker LXC boot2docker grafana
---

## 简介

时间序列数据库，最简单的定义就是数据格式里包含Timestamp字段的数据，比如某一时间环境的温度、CPU使用率等。时间序列数据的更重要的一个属性是如何去查询它，包括数据的过滤，计算等等

influxdb是一个开源的分布式时序、时间和指标数据库，使用go语言编写，它有三个特性

* 时序性（Time Series）：与时间相关的函数的灵活使用（诸如最大、最小、求和等）
* 度量（Metrics）：对实时大量数据进行计算
* 事件（Event）：支持任意的事件数据，换句话说，任意事件的数据我们都可以操作

granfana是一个open source的图形化数据展示工具，可以自定义datasource、自定义报表、显示数据等

## 安装和配置influxdb

我是在Mac OS环境上做的测试，首先是安装docker和influxdb

```
$ brew update

$ brew install boot2docker
$ brew install docker

# 配置 Docker 客户端
$ export DOCKER_HOST=tcp://127.0.0.1:4243

$ brew install influxdb

$ brew install grafana
```

同时说一下influxdb的两个http端口：

* 8083：管理页面端口，访问`http://localhost:8083`可以进入你本机的influxdb管理页面
* 8086：http连接influxdb client端口，一般使用该端口往本机的influxdb读写数据

再说一下grafana的端口，访问http://localhost:3000可以访问本地搭建的grafana，用户名跟密码都是admin

Python3安装influxdb驱动

```
$ pip3 install influxdb
```

## docker使用

`boot2docker init`在VirtualBox中创建一个叫做boot2docker-vm的虚拟机。以后只需要用boot2docker命令来控制这个虚拟机的启动、停止就好了。但是这个命令可能运行报错

![image](../media/image/2017-12-17/01.png)

报错信息是

```
error in run: Failed to initialize machine "boot2docker-vm": exec: "VBoxManage": executable file not found in $PATH
```


## influxdb使用

