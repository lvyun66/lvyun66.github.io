---
title: Composer私有仓库搭建
date: 2017-09-13 19:50:54
tags:
- PHP
- Composer
categories:
- Composer
comments: true
---

## Composer私有仓库搭建

### 资源库

一个资源库是一个包的来源.它是一个 package/versions的列表.Composer将查看所有你定义的repositories以找到你项目所需的资源包.
<!-- more -->
默认情况下已经将`Packagist.org`注册到`composer`.你可以在`composer.json`申明更多的资源库,以便加入你的项目中.

### 资源库类型(`Types`)

```
1. Composer
2. VCS
3. PEAR
4. Package
```
#### 1. Composer

主资源库的类型为`composer`.下面会介绍如何搭建私人仓库.

#### 2. VCS

一旦你有一个包含`composer.json`文件的库存储在线上版本控制系统(如 git或者 svn),你的库就可以被Composer安装.

例如:

```
"repositories": [
    {
        "type": "vcs",
        "url": "https://github.com/yunlvgo/test.git"
    },
    {
        "type": "vcs",
        "url": "svn://192.168.0.1/project/hello/",
        "trunk-path": false,
        "branches-path": false,
        "tags-path": false
    }
],
"config": {
    "secure-http": false		// composer 默认使用 https 协议
},
```

如果 SVN 需要用户密码才能访问可以加入下面代码(与`repositories `同级):

```
{
    "http-basic": {
        "svn://192.168.0.1/": {
            "username": "username",
            "password": "password"
        }
    }
}

```

### 如何搭建私人`composer`仓库

Composer 搭建私人资源库方式:

1. Satis
2. Artifact

这里着重介绍如何使用`satis`搭建.

#### 安装

```
composer create-project composer/satis --stability=dev
```
使用`composer`安装`satis`会比较方便, 可以在当前文件夹下面找到`satis`目录.

#### 配置

`satis`的配置是通过`satis.json`进行的; 在`satis`目录下创建`satis.json`文件, 配置如下:

```
{
    "name": "My Repository",
    "homepage": "http://xxxxx.com",
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/yunlvgo/test.git"
        },
        {
            "type": "vcs",
            "url": "svn://192.168.0.1/project/hello/",
            "trunk-path": false,
            "branches-path": false,
            "tags-path": false
        }
    ],
    "config": {
        "secure-http": false
    },
    "http-basic": {
        "svn://192.168.0.1/": {
            "username": "username",
            "password": "password"
        }
    }
    "require-all": true		//	获取所有的依赖包, 如果要指定使用 require
}
```

我们可以看到这些配置与普通的`composer.josn`几乎一样.

#### 生成仓库

在`satis`目录下运行下面命令:

```
php bin/satis build satis.json public/
```

即可看到会生成一个`public`目录

#### 访问仓库

为了使我们生成的内容可以访问, 我们可以配置 `nginx`指定到`public`目录;或者我们也可以使用php 内置命令`php -S 0.0.0.0:8088 -t public/`

#### 使用

```
"config": {
    "secure-http": false
},
"repositories": [
    {
        "type": "composer",
        "url": "http://192.168.0.1"
    }
],
```
现在我们可以看到`repositories`的`type`是`composer`;
到此我们就可以像正常使用我们的私人仓库了.
