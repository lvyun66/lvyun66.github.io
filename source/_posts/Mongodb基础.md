---
title: MongoDB基础
date: 2017-9-18 23:43:00
tags:
    - mongodb
    - nosql
categories:
    - mongodb
comments: true
---

## MongoDB概念
<!-- more -->

SQL概念 | MongoDB概念| 说明
---|--- | ---
datebases | datebases | 数据库
table | collection | 数据库表/集合
row | document | 数据记录行/文档
column | field | 数据字段/域
index | index | 索引
table joins | | 表连接/MongoDB不支持
primary keys | primary keys | 主键/MongoDB自动设置主键

```
{
    "_id": ObjectId("34asd78guyg971h123g")
    "name": "lvyun"
    "age": 21
}
```


### 数据库

一个MongoDB可以建立多个数据库; 默认数据库为`db`, 存储在`data`目录中

```
//可以显示所有的数据列表
> show dbs
admin   0.000GB
local   0.000GB
spider  0.065GB
```

## CURD

### 查询

```
> db.jianshu.find().limit(1).skip(10).pretty()
{
	"_id" : ObjectId("59b0ee7e4d0fef467045f764"),
	"uid" : "b278f4f88ee9",
	"title" : "我是简小妹",
	"followings" : 44,
	"followers" : 1051,
	"articles" : 8,
	"words" : 4428,
	"likes" : 530,
	"following_url" : "/users/b278f4f88ee9/following",
	"follower_url" : "/users/b278f4f88ee9/followers"
}
```
`limit()`是每次显示的数据条数, 和mysql中的`limit`具有相同的效果
`skip()`偏移量, 和`offset`一个概念
`pretty()`的作用是格式化显示数据

### 插入数据

```
> db.test.insert({'name': 'lvyun', 'age': 21})
WriteResult({ "nInserted" : 1 })
>
> db.test.find().pretty()
{
	"_id" : ObjectId("59c075a13a47c785524653bb"),
	"name" : "lvyun",
	"age" : 21
}
```

## 鉴权

### 创建用户

mongodb创建用户的方式很简单, 通过`createUser`来创建,如果是创建管理用户,先切换`use admin`到`admin`数据库,然后在执行创建命令, 因为`mongodb`是在当前仓库创建用户的.
```
> db.createUser({
...     'user': 'username',
...     'pwd': 'password',
...     'roles': [
...         {
...             'role': 'read',
...             'db': 'admin'
...         }
...     ]
... })
Successfully added user: {
	"user" : "username",
	"roles" : [
		{
			"role" : "read",
			"db" : "admin"
		}
	]
}
```

在`roles`可以指定多个角色,例如:
```
{
    "role" : "userAdminAnyDatabase",
    "db" : "admin"
},
{
    "role" : "dbAdminAnyDatabase",
    "db" : "admin"
},
{
    "role" : "readWriteAnyDatabase",
    "db" : "admin"
}
```

### 数据库角色

要想查看`mongodb`有哪些角色, 可是使用`show roles`
```
> show roles
{
	"role" : "__system",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "backup",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "clusterAdmin",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}

```

这些角色可以分为很多类:

#### 数据库用户角色

> 针对每一个数据库进行控制
```
// 读权限
{
    "role" : "read",
    "db" : "admin",
    "isBuiltin" : true,
    "roles" : [ ],
    "inheritedRoles" : [ ]
}
// 读写权限
{
	"role" : "readWrite",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
```

#### 数据库管理角色

> 针对单独的数据库
```
// 当前数据库的管理者,具有管理权限,但是没有读写权限
{
	"role" : "dbAdmin",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
// 该数据库的拥有者, 具有当前数据库的所有权限
{
	"role" : "dbOwner",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
// 为当前数据库创建, 修改用户角色
{
	"role" : "userAdmin",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}

```

#### 集群管理权限

```
{
	"role" : "clusterAdmin",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "clusterManager",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "clusterMonitor",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "hostManager",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
```
#### 所有数据库管理角色

```
{
	"role" : "dbAdminAnyDatabase",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "readAnyDatabase",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "readWriteAnyDatabase",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "userAdminAnyDatabase",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}

```
#### 超级管理员角色

```
{
	"role" : "root",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
```

由于`root`角色权限过大, 在生产环境中一般不会配置

#### 其他角色

```
{
	"role" : "restore",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "enableSharding",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "__system",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}
{
	"role" : "backup",
	"db" : "admin",
	"isBuiltin" : true,
	"roles" : [ ],
	"inheritedRoles" : [ ]
}

```

> 具体角色可以看[官方文档](https://docs.mongodb.com/manual/reference/built-in-roles/)

**在`mongodb`中,具有`管理权限`的角色不一定具有`读写权限`,如果你只设置了一个用户的`管理权限`, 那么给用户是无法读取`collections`的数据的; 因此在授权的时候如果要给一个用户同时具有管理和读写的权限,那么一定把加上权限.**

### 查看当前db用户

```
> show users
{
	"_id" : "admin.root",
	"user" : "root",
	"db" : "admin",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	]
}
```
