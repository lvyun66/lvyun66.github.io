---
title: Mysql InnoDB锁
date: 2018-7-6 09:40:44
tags:
    - mysql
    - lock
    - innoDB
categories:
    - mysql
comments: true
---
本文主要讨论的是Mysql中InnoDB分支, 所有关于锁的讨论都是基于InnoDB, MyISAM不做讨论.

参考地址： https://dev.mysql.com/doc/refman/5.6/en/innodb-locking.html

## Mysql InnoDB锁的分类
### 共享锁(shared lock)和排它锁(exclusive lock)

InnoDB的共享锁(shared lock)和排它锁(exclusive lock)都实现了标准的行锁。

`共享锁(shared lock)`也就是`S锁`，允许持有S锁的事务读取行数据；
`排它锁(exclusive lock)`也称`X锁`，允许持有X锁的事务更新或删除行数据；

事务T1对行`r`持有S锁，则来自事务T2的处理如下：
1. 如果T2对行申请S锁，则会被立即授权，T1、T2同时对行`r`持有S锁；
2. 如果T2对行申请X锁，则不会被授权。

事务T1对行`r`持有X锁，则事务T2不管申请S锁还是X锁都不会背授权，知道T1释放对`r`的X锁。

### 意向锁(intention lock)

InnoDB支持多粒度锁，允许行锁和表锁共存。例如`LOCK TABLE ... WRITE`等语句在指定的表上采用独占所(X锁)。为什么实现多个粒度级别的锁，InnoDB使用意图锁。意图锁是表级锁定，指示事务稍后对表中的行锁类型(共享S或独占X)。意向锁包括：
1. 意向共享锁(IS)，表示事务将在表中的各个行上设置共享锁；
2. 意向排它锁(IX)，表示事务将在表中的各个行上设置排他锁。

例如： `select ... lock in share mode`设置`IS`锁，`select ... for update`设置`IX`锁。

在事务获取表中行的共享锁之前，它必须先获取表上的IS锁。
在事务获取表中行的排他锁之前，它必须先获取表上的IX锁。

表级类型锁定兼容性：

|T1/T2|  S  |  X  |  IS  |  IX  |
| --- | --- | --- | ---  |  --- |
|  S  |  Y  |  N  |  Y   |   N  |
|  X  |  N  |  N  |  N   |   N  |
|  IS |  Y  |  N  |  Y   |   Y  |
|  IX |  N  |  N  |  Y   |   Y  |

> 说明, Y表示兼容， N表示不兼容

如果事务与现有锁类型兼容，则授权；但是如果与现有锁冲突，则不会被授权。事务一直等待知道冲突的锁类型被释放。如果请求锁类型与现有类型冲突而不能被授权，会导致死锁，则会发生错误。

意向锁不会阻塞出完整表之外的任何内容。意向锁的主要目的是表名某个事物正在锁定的行或者是将要上锁的行。

### 记录锁(record lock)

记录锁是对索引记录的锁定。例如： `select c1 from t where c1 = 10 for update;`防止其他事物插入、更新或删除t.c1 = 10 的行。

即时表没有索引，`记录锁(record lock)`任然会锁定记录。对于这种情况，InnoDB会创建一个隐藏的聚簇索引，并使用此索引进行记录锁定。

### 间隙锁(gap lock)

`间隙锁(gap lock)`是锁定`索引`记录之间的间隙。例如：`select c1 from t where c1 between 10 and 20 for update;`，防止其他事务将10-20之间的值插入到记录中，无论改值是否存在，因为该范围间隙都已经被锁定。

间隙锁可能跨越单个索引值、多个索引值甚至可能为空。

间隙锁是权衡性能和并发的一部分，用于某些事务隔离级别而不是其他级别。

唯一索引锁定不需要使用间隙锁（但是这不包括多列唯一索引的情况有可能会使用到间隙锁）。

### 范围锁(next-key lock)

### 插入意向锁(insert intention lock)

### AOTU-INC锁(AOTU-INC lock)
