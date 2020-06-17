---
layout: post
title: Spring浅谈：声明式事务原理
categories: java之web开发 java之web框架 数据库之mysql
tags: java spring 切面 面向切面编程 AOP AspectJ 面向切面 编译 装饰器设计模式 声明式事务 IoC IoC容器 spring-context spring-aspects pom Maven 增强器 通知方法 代理对象 Cglib代理 事务 数据库 声明式事务 MySQL MyBatis 嵌套事务 @Transactional @EnableTransactionManagement 
---

在上一篇中讲到了使用@Transactional 注解事务运行效果，以及各种嵌套情况下的不同现象，本文就试着去研究一下背后的原理