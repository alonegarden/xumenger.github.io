---
layout: post
title: JDB排查数据库死锁
categories: java之内存管理 深入学习之编译原理 
tags: java JVM jdb 调试 死锁 数据库 Oracle 锁 线程 堆栈 连接池 DBeaver 
---

>工作中遇到的一个小问题，简单记录一下排查过程，没啥技术含量，主要是展示一下jdb 的用法

从模拟器发起撤销请求，每次都会超时，分析日志。发现点击按钮之后，模拟器连续发起多次撤销请求，第一次请求，撤销是成功的，但后续的请求出现加锁超时（撤销会对原纪录加锁）

按照设计，后面的请求也不应该出现加锁超时的情况。幸好这个问题必现，那么我们就可以上手段去排查这个问题

在DBeaver 中尝试对对应撤销的原记录进行加锁，果然加锁超时，那么简单分析一下，可能有这两个原因

1. 出现死锁，那么一定有某个线程被阻塞住了
2. 某个线程在工作完成后，直接退出，没有归还连接
3. 其他原因

思路也很简单，上调试器，当问题出现的时候，去检查对应线程的状态，如果阻塞住了，那么就是死锁了，否则那大概率就是第二个原因了，那就去检查代码是否写的不规范，没有使用异常处理机制，在finally 代码块归还数据库连接

首先以Debug 方式启动Java 进程

>nohup java -server -Xms512m -Xmx2g -XX:PermSize=256m -XX:MaxPermSize=512m -Xdebug -Xrunjdwp:transport=dt_socket,address=7077,server=y,suspend=y -cp /app/server/servercld.jar com.frm.server.api.servercld.CLD_Process_Start com.frm.server.api.serverolt.OLT_Process_Start XUM2 >>/app/log.txt 2>&1 &

然后再重新开启一个Shell，执行下面的命令连接到待调试进程

>jdb -attach localhost:7077

执行[run] 命令，让进程跑起来

通过模拟工具，发起请求，然后发起撤销，果然又出现死锁

等待一段时间后，看日志中有这样的信息

```
2019-09-29 12:03:10.501|WARN 55.13.17.232|8016|CLD8016 - 数据库连接池中有连接长时间未归还（线程名：Worker-thread-1）
2019-09-29 12:03:10.501|WARN 55.13.17.232|8016|CLD8016 - 数据库连接池中有连接长时间未归还（线程名：Worker-thread-1）
```

然后回到调试窗口，执行[threads] 显示所有线程信息，找到线程名是Worker-thread-1 的线程，其线程id 是0xcc4

>所以在设计一个框架的时候，为线程设置有含义的名称是很重要的事情

然后执行[thread 0xcc4] 切换到这个线程，[where] 查看这个线程的调用栈，发现这个线程不是suspend 状态，没有办法看

```
Worker-thread-1[1] where
Current thread isn't suspended.
```

然后执行[suspend 0xcc4] ，挂起当前线程，然后执行[thread 0xcc4] 切换到当前线程，然后查看线程的堆栈

```
> thread 0xcc4
Worker-thread-1[1] suspend 0xcc4
Worker-thread-1[1] where
  [1] java.net.SocketInputStream.socketRead0 (native method)
  [2] java.net.SocketInputStream.socketRead (SocketInputStream.java:116)
  [3] java.net.SocketInputStream.read (SocketInputStream.java:171)
  [4] java.net.SocketInputStream.read (SocketInputStream.java:141)
  [5] oracle.net.ns.Packet.receive (Packet.java:308)
  [6] oracle.net.ns.DataPacket.receive (DataPacket.java:106)
  [7] oracle.net.ns.NetInputStream.getNextPacket (NetInputStream.java:324)
  [8] oracle.net.ns.NetInputStream.read (NetInputStream.java:268)
  [9] oracle.net.ns.NetInputStream.read (NetInputStream.java:190)
  [10] oracle.net.ns.NetInputStream.read (NetInputStream.java:107)
  [11] oracle.jdbc.driver.T4CSocketInputStreamWrapper.readNextPacket (T4CSocketInputStreamWrapper.java:124)
  [12] oracle.jdbc.driver.T4CSocketInputStreamWrapper.read (T4CSocketInputStreamWrapper.java:80)
  [13] oracle.jdbc.driver.T4CMAREngine.unmarshalUB1 (T4CMAREngine.java:1,137)
  [14] oracle.jdbc.driver.T4CTTIfun.receive (T4CTTIfun.java:350)
  [15] oracle.jdbc.driver.T4CTTIfun.doRPC (T4CTTIfun.java:227)
  [16] oracle.jdbc.driver.T4C8Oall.doOALL (T4C8Oall.java:531)
  [17] oracle.jdbc.driver.T4CPreparedStatement.doOall8 (T4CPreparedStatement.java:208)
  [18] oracle.jdbc.driver.T4CPreparedStatement.executeForRows (T4CPreparedStatement.java:1,046)
  [19] oracle.jdbc.driver.OracleStatement.doExecuteWithTimeout (OracleStatement.java:1,336)
  [20] oracle.jdbc.driver.OraclePreparedStatement.executeInternal (OraclePreparedStatement.java:3,613)
  [21] oracle.jdbc.driver.OraclePreparedStatement.executeUpdate (OraclePreparedStatement.java:3,694)
  [22] oracle.jdbc.driver.OraclePreparedStatementWrapper.executeUpdate (OraclePreparedStatementWrapper.java:1,354)
  [23] com.frm.cld.api.clddbc.CLDDBC_DBCnnPool$DelegatedPreparedStatement.executeUpdate (CLDDBC_DBCnnPool.java:1,304)
  [24] com.frm.cld.api.cldhdl.CLDHDL_InternalHdl$CLDHDL_PreparedStatement.executeUpdate (CLDHDL_InternalHdl.java:538)
  [25] com.frm.xum.api.xumado.XUMADO_Cancel._update (XUMADO_Cancel.java:515)
  [26] com.frm.xum.api.xumado.XUMADO_Cancel.update (XUMADO_Cancel.java:381)
  [27] com.frm.xum.wke.XUMTradeCancel.XUMTradeCancel.mainProcess (XUMTradeCancel.java:231)
  [28] com.frm.xum.wke.XUMTradeCancel.XUMTradeCancel.execute (XUMTradeCancel.java:76)
  [29] com.frm.cld.api.cldwkm.CLDWKM_Monitor.execute (CLDWKM_Monitor.java:169)
  [30] com.frm.cld.api.cldwkm.CLDWKM_Monitor.execute (CLDWKM_Monitor.java:64)
  [31] com.frm.cld.api.cldcms.CLDCMS_TCPWrkTask.run (CLDCMS_TCPWrkTask.java:41)
  [32] com.frm.cld.api.cldthd.CLDTHD_ThdExecutor.runWorker (CLDTHD_ThdExecutor.java:1,164)
  [33] com.frm.cld.api.cldthd.CLDTHD_ThdExecutor$Worker.run (CLDTHD_ThdExecutor.java:634)
  [34] java.lang.Thread.run (Thread.java:748)
```

>线程因为死锁不能运行也不是suspend 状态，这时候在Java 中是不能看到其堆栈的，所以先要将其suspend

>正如最开始所说的，这个问题并不一定是线程死锁导致的，也有可能是线程继续运行但是没有归还连接，所以suspend后看到堆栈，并不一定是死锁的堆栈，因为就有可能根本就没有发生死锁，所以可以多次resume 和suspend，如果发现每次suspend后看到的堆栈都是一样的，尤其像这里这样都是在数据库操作、锁的使用等地方，那么很大可能就是死锁了

发现第25行是熟悉的函数，执行[up 24]（对应的是[down 24]）切换到这个栈帧，然后执行[wherei] 查看栈帧

```
Worker-thread-1[1] up 24
Worker-thread-1[25] wherei
  [25] com.frm.xum.api.xumado.XUMADO_Cancel._update (XUMADO_Cancel.java:515), pc = 1,005
  [26] com.frm.xum.api.xumado.XUMADO_Cancel.update (XUMADO_Cancel.java:381), pc = 20
  [27] com.frm.xum.wke.XUMTradeCancel.XUMTradeCancel.mainProcess (XUMTradeCancel.java:231), pc = 706
  [28] com.frm.xum.wke.XUMTradeCancel.XUMTradeCancel.execute (XUMTradeCancel.java:76), pc = 22
  [29] com.frm.cld.api.cldwkm.CLDWKM_Monitor.execute (CLDWKM_Monitor.java:169), pc = 345
  [30] com.frm.cld.api.cldwkm.CLDWKM_Monitor.execute (CLDWKM_Monitor.java:64), pc = 22
  [31] com.frm.cld.api.cldcms.CLDCMS_TCPWrkTask.run (CLDCMS_TCPWrkTask.java:41), pc = 67
  [32] com.frm.cld.api.cldthd.CLDTHD_ThdExecutor.runWorker (CLDTHD_ThdExecutor.java:1,164), pc = 95
  [33] com.frm.cld.api.cldthd.CLDTHD_ThdExecutor$Worker.run (CLDTHD_ThdExecutor.java:634), pc = 5
  [34] java.lang.Thread.run (Thread.java:748), pc = 11
```

然后可以查看这个函数对应的参数信息[dump TradeInfo]

```
Worker-thread-1[25] dump TradeInfo
 TradeInfo = {
    TrxAmt: instance of java.math.BigDecimal(id=4787)
    OrdAmt: instance of java.math.BigDecimal(id=4788)
    TrxDat: 20190929
    ...
    ...
}
```

最后发现果然是实现逻辑的问题，所有的数据库操作都是使用自己申请的连接，但有一个地方不小心又错误的写成使用框架的连接了，错误地混用数据库连接，导致直接死锁！！！！

独立的做一次退货是没有问题的，但是刚好因为模拟器的问题，导致快速、重复发起请求，将这个问题暴露出来了，所以在高并发下的代码果然是稍不注意就有大问题！！！！
