---
title: Mysql创建用户与授权
date: 2017-9-14 20:17:43
tags:
    - mysql
categaries:
    - mysql
comments: true
---
## Mysql创建用户与授权

### 1.连接Mysql

    mysql -h host -u username -p password

    host 主机名或者IP
    username 用户名
    password 用户密码

    注: -p password 之间可以没有空格

### 2. 创建删除用户

    create user 'username'@'host' identified by 'password';

    注: 创建username用户,能从host使用password密码登录

    如果想让用户从任意IP登录,host可设置为%

    drop user 'username'@'%';

    删除用户

### 3. 修改密码

    set password for 'username'@'host' = password('password');

    注: 设置 username 的密码为password

### 4. 授权

    grant all privileges on datebaseName.tableName to 'username'@'host';
    flush privileges;

    设置用户的访问权限, 允许username用户通过host地址访问mysql

    prilileges 包括select, insert , update, delete等权限
    all 代表允许所有的权限

    注: % 代表允许username在任意机器上访问mysql
    如果要设置只允许有个IP地址能访问,则host为具体IP地址,例:192.168.0.1

    *特别声明: grant是增加用户权限,在是原来的基础上增加新权限
    举例说明: 假若dev_00用户现在用select权限, grant insert on *.* to 'dev_00'@'%'
    则dev_00现在用select, insert 权限,如果你在使用grant insert on *.* to 'dev_00'@'%',权限还是select,insert


### 5. 移除授权

    REVOKE privilege ON databasename.tablename FROM 'username'@'host';

    例如: revoke select on *.* from 'dev_00'@'%';flush privileges;

注: 在授权移除授权之后,记得执行flush privileges;刷新权限列表

### 6. 查看用户权限

    show grants for 'username'@'host';

    结果如下:
    GRANT USAGE ON *.* TO 'lvyun'@'%' IDENTIFIED BY PASSWORD '*9FFDA3C5DA0870641C807FC02D75E6461982C3F0'

### 7. 其他说明

    **创建用户的其他方式**

    1. grant select on *.* to 'username'@'%' identified by 'password'

    说明: 创建username用户,可以从任意地址使用password登录mysql,并且授权select

    2. insert into mysql.user(host,user,password) values ('%', 'username', 'password');

    说明: 因为user权限表存放在mysql.user表中,可以直接插入数据到user中(需要用写入user表的权限);不推荐这种方式.


    **grant 选项说明**

    grant select on *.* to 'username'@'%' identified by 'password' with grant option

    其中 with grant option 表示授权用户授权权限,用户能给其他用户授权.(慎用)


### 附录

    privilegs取值

    数据库/数据表/数据列权限：
    Alter: 修改已存在的数据表(例如增加/删除列)和索引
    Create: 建立新的数据库或数据表
    Drop: 删除数据表或数据库
    INDEX: 建立或删除索引
    Insert: 增加表的记录
    Select: 显示/搜索表的记录
    Update: 修改表中已存在的记录
    Delete: 删除表的记录

    全局管理MySQL用户权限：
    file: 在MySQL服务器上读写文件。
    PROCESS: 显示或杀死属于其它用户的服务线程。
    RELOAD: 重载访问控制表，刷新日志等。
    SHUTDOWN: 关闭MySQL服务。

    特别的权限：
    ALL: 允许做任何事(和root一样)。
    USAGE: 只允许登录--其它什么也不允许做。
