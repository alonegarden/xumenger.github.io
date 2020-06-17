---
layout: post
title: Spring浅谈：生命式事务
categories: java之web开发 java之web框架 数据库之mysql
tags: java spring 切面 面向切面编程 AOP AspectJ 面向切面 编译 装饰器设计模式 声明式事务 IoC IoC容器 spring-context spring-aspects pom Maven 增强器 通知方法 代理对象 Cglib代理 事务 数据库 声明式事务 MySQL MyBatis 
---

## 准备环境

先来简单回顾一下Mac 上MySQL 的相关命令

```shell
// 启动MySQL 服务器
$ mysql.server start

// 登录MySQL 服务器
$ mysql -uroot -p
```

接下来的声明式事务的测试都是基于MyBatis 框架，所以先在pom.xml 中引入需要依赖

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>5.1.15.RELEASE</version>
</dependency>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aspects</artifactId>
  <version>5.1.15.RELEASE</version>
</dependency>
```

## 准备一个测试案例

