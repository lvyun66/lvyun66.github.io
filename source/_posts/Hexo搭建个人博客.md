---
title: Hexo搭建个人博客
date: 2017-09-14 20:28:55
tags: Hexo
categories: Hexo
comments: true
---

## Hexo搭建个人博客

### 什么是Hexo?

Hexo是一个用Node.js写的一个博客框架, 支持Markdown语法, 支持自定义主题, 同时可以在几秒钟之内生成静态页面.
<!-- more -->
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

安装完成之后, 可以得到下面的这个目录结构:

```
|-- _config.yml
|-- db.json
|-- node_modules
|-- package-lock.json
|-- package.json
|-- scaffolds
|-- source
`-- themes
```
`_config.yml` 是网站的配置文件, 在后面会介绍;
`package.json` 是应用程序信息, 你可以随时移除里面的应用;因为我额外安装了几个应用, 因此可能会和初始化安装的不一样;
```
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.3.9"
  },
  "dependencies": {
    "gitment": "0.0.3",
    "hexo": "^3.2.0",
    "hexo-deployer-git": "^0.3.1",
    "hexo-deployer-heroku": "^0.1.2",
    "hexo-deployer-openshift": "^0.1.2",
    "hexo-deployer-rsync": "^0.1.3",
    "hexo-generator-archive": "^0.1.4",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-feed": "^1.2.2",
    "hexo-generator-index": "^0.2.1",
    "hexo-generator-sitemap": "^1.2.0",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.3.0",
    "hexo-renderer-marked": "^0.3.0",
    "hexo-renderer-stylus": "^0.3.3",
    "hexo-server": "^0.2.2",
    "hexo-util": "^0.6.1"
  }
}
```

`themes` 是主题文件夹, `Hexo`会根据主题生生成不同的静态文件;

### 配置网站信息

`Hexo`大部分的配置都包含在`_confog.yml`文件中, 我们着重配置下面这些信息, 其他的都是用默认值; 如果有兴趣可以自己去研究其他配置的用法
```
# Site
title: 夏目的友人帐是空的-Record
subtitle:
description:
author: lvyun
language: zh-Hans
timezone:
```

如果我们要使用主题, 可以在配置中添加下面信息:
```
theme: next
```

### 生成, 部署

经过上面的配置, 接下来就是生成静态文件和部署了

```
$ hexo clean
$ hexo g
$ hexo d
```

然后启动`Hexo`默认服务器

```
$ hexo server
```

通过访问`http://localhost:4000/`来访问自己的博客了.

到此, 本地搭建的Hexo就完成了
