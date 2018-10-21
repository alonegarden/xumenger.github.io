---
layout: post
title: 分布式事务
categories: 大型系统架构 
tags: 分布式事务 TCC TCC-Transaction 事务 数据库 ACID 数据一致性 注解 柔性事务 刚性事务 CAP 
---

在一个软件系统中，最重要的是什么？当然是数据，数据必须是可信的，为了保证数据的一致性，在一个简单的单体应用中，我们假如用到了多线程并发机制，那么必须在多线程竞争的数据上使用竞态条件来保证线程安全，最根本的其实就是为了保证数据的一致性！

那如果是一个单体应用需要外部存储的时候，现代数据库管理系统为我们提供了很好的选择，数据库提供的事务机制（ACID）可以有效的保证数据的一致性！

考虑这样一个场景，假如在数据库提交事务的时候突然断电，那么它是怎么恢复的呢？

以SQL Server 为例！

SQL Server 数据库是由两个文件组成的，一个数据库文件和一个日志文件，通常情况下，日志文件要比数据库文件大得多。数据库进行任何写操作的时候都要先写日志文件，再写数据文件

同样的道理，在执行事务的时候，数据库首先记录下这个事务的redo 操作日志，然后才开始真正操作数据库，在操作之前首先会把日志文件写入磁盘，那么突然断电的时候，即使操作没有完成，在重启数据库的时候，数据库会根据当前数据的情况进行undo 回滚或者redo 重做，这样就可以保证数据的强一致性

OK，在单体应用中可以通过数据库的ACID 事务机制来保证数据强一致性

但如果是在一个分布式系统中做开发，尤其是在一个微服务架构的系统中，一个业务操作可能分布在多台机器、多个应用中，当中任何一步的失败都会导致整个业务操作的失败，这个时候如何保证数据的一致性呢？！

答案就是分布式事务！

## 分布式事务

在[分布式计算的八大谬论](http://www.xumenger.com/the-eight-fallacies-of-distributed-computing-20180817/)中有提到过分布式系统的难点，即分布式的网络环境很复杂

在分布式系统中可能的异常有：某个机器宕机、网络异常、消息丢失、消息乱序、数据错误、不可靠的TCP、存储数据丢失等等异常



## TCC-Transaction

>[TCC-Transaction](https://github.com/changmingxie/tcc-transaction)是一个开源的TCC 分布式事务框架

>[《tcc-transaction 官方文档 —— 使用指南1.2.x》](https://github.com/changmingxie/tcc-transaction/wiki/%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%971.2.x)

在TCC 中，一个事务可以包含多个操作。TCC-Transaction 将每个业务操作抽象成事务参与者，每个事务可以包含多个参与者

参与者需要声明try/confirm/cancel 三个类型的方法，和TCC 的操作一一对应。在程序中，通过@Compensable 注解标记在try 方法上，并填写对应的confirm/cancel 方法

```java
public class CapitalTradeOrderServiceImpl implements CapitalTradeOrderService {

    // try
    @Comparable(confirmMethod = "confirmRecord", cancelMethod = "cancelRecord", transactionContextEditor = MethodTransactionContextEditor.class)
    public String record(TransactionContext transactionContext, CapitalTradeOrderDto tradeOrderDto) 
    {

    }

    // confirm
    public void confirmRecord(TransactionContext transactionContext, CapitalTradeOrderDto tradeOrderDto) 
    {

    }

    // cancel
    public void cancelRecord(TransactionContext transactionContext, CapitalTradeOrderDto tradeOrderDto) 
    {

    }

}
```

## 参考资料

* [聊聊分布式事务，再说说解决方案](https://www.cnblogs.com/savorboard/p/distributed-system-transaction-consistency.html)
* [tcc-transaction 官方文档 —— 使用指南1.2.x](https://github.com/changmingxie/tcc-transaction/wiki/%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%971.2.x)