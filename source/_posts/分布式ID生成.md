---
title: 分布式ID生成方案
date: 2018-09-18 14:40:54
tags:
    - ID Generator
categories:
    - 其他
comments: true
---

## Mysql ID auto_increment
利用Mysql递增特性(auto_increment), 保证ID的全局唯一性.

```sql
alter table test AUTO_INCREMENT = 1000;
set auto_increment_offset = 1;
```

Pros:
1. 简单
2. 可控

Cons:
1. 无法应对高并发, 数据库压力大

## UUID

通过本地程序服务生成, 不用远程调用

Pros:
1. 本地生成, 不需要远程调用
2. 全局位移不重复
3. 水平扩展能力强

Cons:
1. 占用空间大, 占用128bits,通过存储类型为字符串, 索引效率低
2. 无法保证ID的持续递增性

## Redis ID/Mongo ObjectID
当数据库性能不够时，我们可以使用redis生成ID，这主要依赖redis单线程的特性，用redis的原子操作incr和ncrby来实现。
如果需要在同一时间生成更多的ID，可以使用Redis集群。

Pros：
1. 不依赖数据库，性能比数据库好
2. ID是数字且有序，对于排序有很大的帮助

Cons：
1. 如果系统中，本身没有redis，需要增加新组件，增加系统复杂度
2. 需要进行编码

## Flickr
同样是利用Mysql的`auto increment`机制，不过与第一种方案不同在于，这个方案新增一张表，来存在ID
```sql
Field              Type              Collation        Null    Key     Default  Extra           Privileges            Comment
-----------------  ----------------  ---------------  ------  ------  -------  --------------  --------------------  -------------------------
id                 char(1)           utf8_general_ci  NO      PRI     (NULL)   auto_increment  select,insert,update  unique key
stub               char(1)           utf8_general_ci  NO      UNI     (NULL)                   select,insert,update
updated_time       int(10) unsigned  (NULL)           YES             (NULL)                   select,insert,update  最近更新时间，采用UTC时间戳
```
使用`auto_increment + replace into + MyISAM`来实现ID生成服务。

同样，当性能不足，可以使用Mysql集群，设置auto_increment的起始值和increment_offset偏移量即可生成全局唯一ID。

Pros：
1. ID可控，简单

Cons：
1. 写可能成为瓶颈（但只有机器足够多好像也不是问题）
2. 如果采用Mysql集群，则可能导致在一定时间内ID不连续

## SnowFlake
Twitter的开源的SnowFlake ID生成算法，
41位的时间序列（精确到毫秒）
10位的机器标识
12位的计数顺序号（最多毫秒生成4096个ID序列）

Pros：
1. 高性能，低延迟，独立的应用
2. 按时间排序

Cons：
1. 需要独立部署服务或者依赖zookeeper服务

## Leaf
Leaf是美团的分布式ID生成服务，可以认为是Flickr的变种。不再基于Mysql的auto_increment特性，批量生成一批ID。
max_id表示当前生成的最大ID，用ID用完再去获取新的ID号段，这样可以很大程度降低数据库的压力。
同时还可以用biz_tag来区分不同的业务，每个业务之间相互不影响。

Pros：
1. 性能好，ID顺序生成

Cons：
1. 需要单独开发一套服务，成本较大

## Other
例如订单号的生成： 日期+用户
