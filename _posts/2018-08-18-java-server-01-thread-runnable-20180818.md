---
layout: post
title: Java服务端开发：Thread与Runnable
categories: 大型系统架构 java之多线程 java之面向对象
tags: Java服务端开发 Java 服务端 服务器 Thread Runnable Delphi C++ Windows JVM 类 接口 Override 
---

之前使用Delphi、C++ 在Windows 下做开发，总结了很多[多线程](http://www.xumenger.com/tags/#%E5%A4%9A%E7%BA%BF%E7%A8%8B)的文章，运行原理不再多言，因为现在要用到Java 做服务端开发，也要用到多线程的东西，所以还是写一篇流水账文章把相关的API 梳理一遍吧

可以说这个【Java服务端开发】系列的文章就是一系列的流水账文章，更多的是介绍API 怎么用的，虽然可以参考[官方网站](http://www.oracle.com/technetwork/cn/java/index.html)和[官方文档](https://docs.oracle.com/en/java/)。我还是写下来是因为选择其中常用的点，以后倒方便随时翻阅参考！

继续说Java 下的多线程！Java 中创建线程有两种方法：

## Thread类

继承Thread 类，重写Thread 的run() 方法

```java
class AThread extends Thread{
    public synchronized void run(){
        for(int i=0; i<5; i++){
            System.out.println(Thread.currentThread().getName());
            try{
                Thread.sleep(100);
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args){
        AThread t1 = new AThread();
        AThread t2 = new AThread();
        t1.start();
        t2.start();
    }
}
```

编译运行的效果如下

![](../media/image/2018-08-18/01-01.png)

## Runnable接口

实现Runnable 接口，实例化Thread 类

```java
class Runner implements Runnable{
    @Override
    public synchronized void run(){
        for(int i=0; i<5; i++){
            System.out.println(Thread.currentThread().getName());
            try{
                Thread.sleep(100);
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
    }


    public static void main(String[] args){
        Runner r = new Runner();
        Thread t1 = new Thread(r, "thread-1");
        Thread t2 = new Thread(r, "thread-2");
        t1.start();
        t2.start();
    }
}
```

![](../media/image/2018-08-18/01-02.png)