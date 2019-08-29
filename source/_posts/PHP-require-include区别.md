---
title: PHP中require和include区别？
date: 2019-08-29 09:29:00
tags:
- PHP
categories:
- PHP
comments: true
---

在Web项目中，include和require是模块化设计的基础，当然现在我们我们都是直接使用Composer来管理模块之间的相互依赖，但spl_autoload_register底层的设计中依然用到include和require。

被包含文件先按照参数的给出的路径寻找，如果没有给出目录（只有文件名）时按照include_path指定的目录寻找。如果在include_path下没有找到文件则include最后才在调用脚本文件所在的目录和当前工作目录下寻找。

<!-- more -->

**include和require没有本质上的区别，唯一的区别在于错误级别上，当文件无法被正常加载时include会抛出warming，而require会抛出error错误。**

## include的基本用法及特点

1. 被包含的文件将继承include所在行具有的全部变量范围，比如调用文件前面定义了一些变量，那么这些变量就能够在被包含的文件中使用，反之，被包含文件中定义的变量也将从include调用初开始被调用文件使用；
2. 被包含文件中定义的函数、类在include执行之后将可以被随处调用，即具有全局作用域；
3. include是在运行时加载文件并执行，而不是在编译阶段。

**include是一个特殊的语言结构，其参数不需要括号。**

## require的基本用户及特点

用法基本与include一致，除了处理失败的处理方式不同之外。

## include_once和require_once

`include_once`语句和`include`语句完全相同，唯一区别是PHP会检查该文件是否已经被包含过，如果是则不会再次包含。

`require_once`和`require`也是同理。


> 最后引用鸟哥的一篇文章 [再一次, 不要使用(include/require)_once](http://www.laruence.com/2012/09/12/2765.html)，我们应该尽量少使用(include/require)_once

## 引用文章
> [PHP7内核剖析](https://www.kancloud.cn/nickbai/php7/363301)
> [再一次, 不要使用(include/require)_once](http://www.laruence.com/2012/09/12/2765.html)

## Thinks

感谢你的阅读。
