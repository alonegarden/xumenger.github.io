---
layout: post
title: MySQL/Oracle执行计划
categories: 数据库之mysql 数据库之sql 数据库之oracle
tags: 数据库 关系型数据库 SQLServer Oracle MySQL SQL 数据库引擎 执行计划 Linux 
---

MySQL 查看SQL 的执行计划

```sql
explain

select users.*
from users 
where id = 
(   select user_id 
    from blogs
    where id = 
    (   select comments.blog_id
        from comments
        where comments.id = '1'
    )
);
```

Oracle 查看SQL 的执行计划

```sql
explain paln for

select users.*
from users 
where id = 
(   select user_id 
    from blogs
    where id = 
    (   select comments.blog_id
        from comments
        where comments.id = '1'
    )
);

commit;
select * from table(dbms_xplan.display);
```
