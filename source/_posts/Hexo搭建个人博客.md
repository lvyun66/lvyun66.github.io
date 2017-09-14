---
title: Hexo搭建个人博客
date: 2017-9-14 20:28:55
tags: Hexo
categories: Hexo
comments: true
---

## Hexo搭建个人博客

### 什么是Hexo?

Hexo是一个用Node.js写的一个博客框架, 支持Markdown语法, 支持自定义主题, 同时可以在几秒钟之内生成静态页面.

### 安装

#### 依赖

安装`Hexo`, 需要依赖`Git`和`Node.js`

`Linux` 默认安装了`Git`,因此我们不需要单独在安装; 如果嫌弃系统默认安装安装的`Git`版本太低可以参考一下我的文章

接下来就是安装`Node.js`了, 我这里使用的是源码安装; 如果觉得麻烦, 可以直接使用`yum`来安装, 或者其他的方法.

```
// 准备编码安装的工具
$ yum -y install gcc make gcc-c++ openssl-devel wget
// 下载源码
$ wget https://nodejs.org/dist/v6.11.3/node-v6.11.3.tar.gz
$ tar -xvzf node-v6.11.3.tar.gz
$ cd node-v6.11.3
$ ./configure --prefix=/usr/local/nodejs
$ make
$ make install
// 验证安装是否成功
$ node -v
```

使用 `nvm`安装`nodejs`

```
$ wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
$ nvm install stable
```

#### 安装Hexo

````
$ npm install -g hexo-cli
````

这里附上 [Hexo](https://hexo.io/zh-cn/docs/) 官网地址, 如果有什么不清楚的可以直接去官网或者在文章底部提`Issue`.

### 初始化

Hexo的初始化很简单, 直接使用`init`命令即可;

```
// hexo init <folder>
$ hexo init /var/www/hexo
$ cd /var/www/hexo
$ npm install
```

如果没有目录`/var/www/`, 可以先新建`mkdir -p /var/www`;
