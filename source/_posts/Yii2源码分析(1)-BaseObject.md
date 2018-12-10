---
title: Yii2源码分析(1)-BaseObject
date: 2018-12-01 19:18:18
tags:
    - Yii2
    - PHP
categories:
    - Yii2
comments: true
---

`BaseObject`实现了Yii2最基本的特性`属性`，使得对象的成员变量具有可控性。

一个`属性`是具有权限控制的成员变量，读或写通过`getter`和`setter`方法实现。如果属性没有实现`getter`或者`setter`，那么代表这个属性没有读写的权限，不能对该属性进行任何的操作，如果只有`getter`或`setter`方法，表示属性只读或者只写。
