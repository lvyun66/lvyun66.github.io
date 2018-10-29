---
title: Mysql Truncate、Delete区别
date: 2018-10-29 10:30:56
tags:
    - Mysql
categories:
    - Mysql
comments: true
---

上次在使用`Truncate`和`Delete`的时候，被问到两者之间的区别，结果一脸蒙蔽，不知所以然。

只是大概的回答了下两者之间的区别，而没有说到根本区别在哪？今天抽空在网上查找了两者之间的区别，刚好填补上之前的空缺。

## 共同点
```text
1. 两者皆可以清空数据表；
2. 两者与 TRANSACTION， 都可以回滚；
```


## 不同点

**DELETE**
```text
1. DELETE是 DML 命令;
2. DELETE使用行锁执行，表的中每一行被锁定后执行删除操作；
3. 会激活触发器；
4. 会生成事务，可以rollback。
```

**TRUNCATE**
```text
1. TRUNCATE 是 DLL 命令；
2. 如果 PK 是自增，将会重置计数器
```

附上`Stackover Flow`上的一个精彩回答，戳此可直达  [comparison-of-truncate-vs-delete-in-mysql-sqlserver](https://stackoverflow.com/questions/20559893/comparison-of-truncate-vs-delete-in-mysql-sqlserver)

## 什么 DDL、DML、DCL？

## DDL
`数据定义语言（data definition language）`，是对数据库对象中各类对象的定义。主要包括用户、库、表、视图、索引、触发器、事件、存储过程已经函数等。

**关键词：**
`create`、`drop`、`alter`和`truncate`等语句关键词。

## DML
`数据操作语言（data manipulation language）`，用于添加、删除、更新和查询数据库记录，并检查数据完整性。

**关键词：**
insert、delete、update和select

## DCL
`数据控制语言（data control language）`，用于控制不同数据段的直接许可和访问级别。DCL定义了数据库、表、字段、用户的访问权限和安全级别。

**关键词：**
`grant`、`revoke`

## 暂时写这么多，后面会把具体的用例补上。
