---
layout: post
title: 数据库隔离机制
categories: 数据库之sql 数据库之oracle 数据库之mysql
tags: 数据库 sql oracle mysql 隔离机制 事务 DBeaver SQLDeveloper 幻读 不可重复读 脏读
---

数据库特性：原子性、一致性、隔离性、持久性，数据库的隔离机制是属于隔离性的范畴

先看一个Oracle 下的例子（Oracle 默认的事务隔离级别是READ COMMITTED，且该例子需要设置不自动提交事务）

**在DBeaver下打开两个Tab 页按照下面的顺序执行**

* “事务1” insert，不进行commit
* “事务1” select，可以查到记录
* “事务2” select，可以查到记录
* “事务2” select for update，也可以成功，不太理解了
* “事务2” insert，直接报错，唯一键冲突
* “事务1” rollback
* “事务2” 再insert，这时候可以成功

>排查发现DBeaver，开启的两个Tab 页，并不是每个Tab 页下对应一个会话，而是共用一个会话

**为了模拟两个会话，使用一个DBeaver，一个SQL Developer**

* 事务1 insert，不进行commit
* 事务1 select，可以查到记录
* 事务2 select，查不到记录
* 事务2 select for update，查不到记录
* 事务2 insert，直接阻塞，注意是阻塞，而不是报唯一键冲突
* 事务1 rollback
* 事务2 从上面阻塞的insert 中出来，成功insert 数据

>再强调一下，这个是在READ COMMITTED 隔离级别下的执行效果

## 事务隔离机制

在没有数据库隔离性的情况下，多用户并发操作可能会发生的问题

**脏读**

脏读是指一个事务读取了未提交事务执行过程中的数据。当一个事务的操作正在多次修改数据，而在事务还未提交的时候，另外一个并发事务来读取了数据，就会导致读取到的数据并非是最终持久化之后的数据，这个数据就是脏读的数据。最典型的例子就是银行转账，从A账户转账100 到B 账户，脚本命令为

```sql
update account set money = money + 100 where username = 'B';

update account set money = money - 100 where username = 'A';
```

在这个事务执行过程中，另外一个事务读取结果发现B 账户中的钱已经到账，提示B 钱已到账，B 就进行了下一步的操作。但最终转账事务失败，导致操作回滚。实际上B 并未收到钱，但进行了下一步的操作，造成了损失，这就是脏读

**不可重复读**

不可重复读是指对于数据库中的某个数据，一个事务执行过程中多次查询返回不同查询结果，这就是在事务执行过程中，数据被其他事务提交修改了

不可重复读同脏读的区别在于，脏读是一个事务读取了另一未完成的事务执行过程中的数据，而不可重复读是一个事务执行过程中，另一事务提交并修改了当前事务正在读取的数据

**虚读(幻读、幻行)**

幻读是事务非独立执行时发生的一种现象，例如事务T1 批量对一个表中某一列列值为1 的数据修改为2 的变更，但在这时，事务T2 对这张表插入了一条列值为1 数据，并完成提交。此时，如果事务T1 查看刚刚完成操作的数据，发现还有一条列值为1 的数据没有进行修改，而这条数据其实是T2 刚刚提交插入的，这就是幻读

幻读和不可重复读都是读取了另一条已经提交的事务（这点同脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）

针对以上可能出现的并发问题，数据库提供不同的**事务隔离级别**

* READ UNCIMMITTED（未提交读）。事务中的修改，即使没有提交，其他事务也可以看得到，也就是脏读，这种隔离级别会引起很多问题，如无必要，不要随便使用
* READ COMMITTED（提交读）。是大多数据库系统的默认隔离级别，这种隔离级别就是一个事务的开始，只能看到已经完成的事务的结果，正在执行的，是无法被其他事务看到的，但是其他事务执行完成Commit 后还是会对当前事务产生影响，会存在不可重复读问题
* REPEATABLE READ（可重复读）。可以解决脏读、不可重复读的问题，该级别保证了每行的记录的结果是一致的，也就是上面说的读了旧数据（不可重复读）的问题，但却无法解决另一个问题，幻行，顾名思义就是突然蹦出来的行数据。指的就是某个事务在读取某个范围的数据，但另一个事务又向这个范围的数据去插入数据，导致多次读取的时候，数据的行数不一致
* SERIALIZABLE（可串行化）。这是最高的隔离级别，它通过强制事务串行执行（注意是串行），避免了前面的幻读情况，但由于大量加上锁，导致大量的请求超时，因此性能会比较低，在特别需要数据一致性且并发量不需要那么大的时候才可能考虑这个隔离级别

>应用程序的设计开发者及数据库管理员可以依据应用程序的需求及系统负载（workload）而为不同的事务选择不同的隔离级别（isolation level）

## Oracle的事务隔离机制

Oracle 数据库支持READ COMMITTED 和SERIALIZABLE 这两种事务隔离级别，默认系统事务隔离级别是READ COMMITTED，也就是读已提交

执行下面的SQL 可以查看系统默认事务隔离级别，也是当前会话隔离级别

```sql
--首先创建一个事务
declare
    trans_id VARCHAR2(100);
begin
    trans_id := dbms_transaction.local_transaction_id( TRUE );
end;


--查看事务隔离级别
select s.sid, s.serial#,
    case BITAND(t.flag, POWER(2, 28))
        when 0 then 'READ COMMITTED'
        else 'SERIALIZABLE'
    end as isolation_level
from v$transaction t
join v$session s on t.addr = s.taddr and s.sid = sys_context('USERENV', 'SID');
```

可以在事务开始时使用以下语句设定事务的隔离级别

* 已提交读模式: SET TRANSACTION ISOLATION LEVEL ＝ READ COMMITTED;
* 串行模式: SET TRANSACTION ISOLATION LEVEL ＝ SERIALIZABLE;
* 只读模式: SET TRANSACTION ＝ READ ONLY;

## MySQL的事务隔离机制

MySQL支持这些事务隔离级别，默认是REPEATABLE READ，这个和Oracle 默认的隔离级别不同

* READ UNCOMMITTED
* READ COMMITTED
* REPEATABLE READ
* SERIALIZABLE

下面是在MySQL 下与数据库隔离级别相关的命令

```sql
--查看当前会话隔离级别
select @@tx_isolation;


--查看系统当前隔离级别
select @@global.tx_isolation;


--设置当前会话隔离级别
set session transaction isolatin level repeatable read;


--设置系统当前隔离级别
set global transaction isolation level repeatable read;
```

## 参考资料

* [数据库四大特性及数据库隔离级别](https://blog.csdn.net/sinat_35322593/article/details/81040479)
* [Oracle事务隔离级别](https://blog.csdn.net/leozhou13/article/details/50449965)
* [数据库的四种隔离级别](https://www.cnblogs.com/s-b-b/p/5845096.html)