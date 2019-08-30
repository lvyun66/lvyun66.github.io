---
title: Composer语义化版本
date: 2018-10-09 15:55:42
tags:
- PHP
- Composer
categaries:
- Composer
comments: true
---

## 什么是语义化版本？

版本格式：主版本号.次版本号.修订号，版本号递增规则如下：
<!-- more -->
1. 主版本号：当你做了不兼容的 API 修改，
2. 次版本号：当你做了向下兼容的功能性新增，
3. 修订号：当你做了向下兼容的问题修正。

先行版本号及版本编译信息可以加到“主版本号.次版本号.修订号”的后面，作为延伸。

详情见[语义化版本2.0.0](https://semver.org/lang/zh-CN/)

## Composer版本限制
