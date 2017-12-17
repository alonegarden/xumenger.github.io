---
layout: post
title: 时序数据库influxdb
categories: 数据库之influxdb python之数据库编程
tags: 数据库 时序数据库 influxdb python python3 docker LXC boot2docker grafana
---

## 安装和配置influxdb

我是在Mac OS环境上做的测试，首先是安装docker和influxdb

```
$ brew update

$ brew install boot2docker
$ brew install docker

# 配置 Docker 客户端
$ export DOCKER_HOST=tcp://127.0.0.1:4243

$ brew install influxdb
```

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



## InfluxDB和Docker简介

InfluxDB是一个开源的时序数据库，使用GO语言开发，特别适合用于处理和分析资源监控数据这种时序相关数据。而InfluxDB自带的各种特殊函数如求标准差，随机取样数据，统计数据变化比等，使数据统计和实时分析变得十分方便。在我们的容器资源监控系统中，就采用了InfluxDB存储cadvisor的监控数据

Docker是一个开源的引擎，可以轻松地为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括虚拟机、OpenStack集群和其他的基础应用平台

Docker通常用于如下场景：

* web应用的自动化打包和发布
* 自动化测试和持续集成、发布
* 在服务型环境中部署和调整数据库或其它的后台应用
* 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境

云主机可以选择操作系统镜像快速创建主机，这笔虚拟机更便捷了，我们本地也可以通过docker这么做。它依赖于LXC（Linux Container），能从网络上获得配置好的Linux镜像，非常容易地在隔离的系统中运行自己的应用。也因为底层是一个LXC，所以在Mac OS X下需要在VirtualBox中跑一个精小的LXC(这里是一个 Tiny Core Linux，完全在内存中运行，个头只约 24MB，启动时间小于 5 秒的 boot2docker) 虚拟机，构建在VirtualBox中。以后的通信过程就是【docker --> boot2docker --> container】，端口或磁盘映射也是遵照这一关系