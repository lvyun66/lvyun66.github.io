---
title: Composer升级库
date: 2017-12-28 14:38:50
tags:
    - PHP
    - Composer
categories:
    - Composer
comments: true
---

## Composer升级库

### required

增加一个新的依赖包到当前目录的`composer.json`，

```
composer require xxxx/xxx:tag
```

在新增或者更新依赖时，修改后的依赖关系将被安装和更新。

### install

从当前目录的`composer.json`读取，处理依赖关系并且安装到当前目录的`vendor`目录下。

```
composer install
```

如果目录下存在`composer.lock`文件，则会从此文件读取依赖。这样做的原因的是确保每个使用者都能使用相同的依赖库，保证环境的一致性。

### update

获取依赖的最新最新版本或者指定版本信息，并且更新`composer.lock`，但是不会更新`composer.json`文件。

### 三者之间的区别
`update`，会根据语义化版本号来升级，例如如果`composer.json`定义为`^8.0`，则会自动升级到`8.0-9.0`之间最新的版本。PS：`league/csv`的V8版本中最高为`v8.2.2`。

```text
[root@b1 yesdatphp]# composer update league/csv

Do not run Composer as root/super user! See https://getcomposer.org/root for details
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 0 installs, 1 update, 0 removals
  - Updating league/csv (8.2.1 => 8.2.2): Downloading (100%)
Writing lock file
Generating autoload files
```

当最新的版本号不在`composer.json`声明的语义化版本范围时，使用`composer update`会无效。例如`filsh/yii2-oauth2-server`在`composer.json`中的定义为`"filsh/yii2-oauth2-server": "v1.0"`，直接使用`composer update`升级会失败。

```text
[root@b1 yesdatphp]# composer update filsh/yii2-oauth2-server

Do not run Composer as root/super user! See https://getcomposer.org/root for details
Loading composer repositories with package information
Updating dependencies (including require-dev)
Nothing to install or update
Generating autoload files
```

要想升级到最新的版本，我们需要重新声明依赖，这样会覆盖`composer.json`已经声明的依赖，并且在安装完成之后同步更新`composer.lock`

```
composer require filsh/yii2-oauth2-server:~2.0
```
这样会自动安装最新的依赖库到vendor下面。

## 总结
使用`composer`升级库：
1. `update` 只会更新到`composer.json`定义的语义化版本的范围内最新的版本，如果超过范围或者没有，这不会做更新；
2. `require` 当更新大版本时或者是超出了定义的版本时，使用`composer require`。
