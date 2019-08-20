---
title: PHP如何获取客户端真实IP?
date: 2019-08-20 19:34:08
tags:
- PHP
- HTTP
categories:
- PHP
comments: true
---

PHP获取客户端IP的操作很常见，记录用户登录IP、根据IP判断用户位置、IP白名单都是一些基本的业务需求。因此如何获取准确的IP地址也是这些需求的必要前提。

<!-- more -->

## HTTP
说到IP，就和HTTP协议离不开，HTTP header中和IP相关的部分包括：`X-Forwarded-For`、`via`、`X-Real-IP`等。

### X-Forwarded-For

`X-Forwarded-For`是一个HTTP扩展头部，在HTTP/1.1(RFC 2616)中并没有对它的定义，是之后才被写入[RFC 7239 (Forwarded HTTP Extension)](https://tools.ietf.org/html/rfc7239)标准中的。

XFF请求头格式如下：
```
X-Forwarded-For: client, proxy1, proxy2
```

经过多个代理服务器后，如果严格安装XFF标准，服务端最终会收到以下信息：
```
X-Forwarded-For: client, IP1, IP2, IP3
```

也就是说代理服务器会把上一个代理服务器的IP地址追加到`X-Forwarded-For`后面，因此`X-Forwarded-For`的第一个值就是客户端的真正IP。由于`X-Forwarded-For`很容易被篡改，所有获取的的客户端IP不是真实存在的。

## NGINX

接下来我们了解下NGINX作为代理服务器时设置的一些参数，`HTTP-X-FORWARDED-FOR`、`X-REAL-IP`、`REMOTE-ADDR`。


### HTTP-X-FORWARDED-FOR
`HTTP-X-FORWARDED-FOR`就是HTTP协议中的`X-Forwarded-For`，在设置`HTTP-X-FORWARDED-FOR`有多重方式：
```config
proxy_set_header HTTP-X-FORWARDED-FOR $http_x_forwarded_for;
// 将上一个的HTTP-X-FORWARDED-FOR值复制给当前HTTP-X-FORWARDED-FOR，即忽略上一次代理的转发

proxy_set_header HTTP-X-FORWARDED-FOR $remote_addr;
// HTTP-X-FORWARDED-FOR被赋值为上一个代理服务器的IP

proxy_set_header HTTP-X-FORWARDED-FOR $proxy_add_x_forwarded_for;
// $proxy_add_x_forwarded_for = $http_x_forwarded_for + $remote_addr
```

### X-REAL-IP
是自定义HTTP header，需要在服务端自行实现该头部协议。
```config
proxy_set_header X-REAL-IP $remote_addr;
```

### REMOTE-ADDR
因为HTTP协议是应用层协议，因此header中的信息都是可以被篡改，都是不安全的。

`REMOTE-ADDR`是解析TCP/IP协议中的IP地址，如果这个值不正确，TCP/IP就不会建立连接，这个值是安全可靠的，我们可以通过这个值获取上一个代理服务器的真实IP。

## PHP获取客户端IP
在PHP中获取客户端IP可以通过以下的三种方式：

```php
$_SERVER['REMOTE-ADDR'];
$_SERVER['HTTP-X-FORWARDED-FOR'];
$_SERVER['HTTP-REAL-IP'];
```

如果后端服务没有通过nginx做代理直接连接，`$_SERVER['REMOTE-ADDR']`可以直接获取客户端的真实IP；但如果使用了代理，则获取到就是代理服务器的IP。

使用NGINX等web server进行反向代理时，在nginx配置正确的情况下，要使用`$_SERVER['HTTP-X-FORWARDED-FOR']`或`$_SERVER['HTTP-REAL-IP']`获取客户端真实IP。

## 结论

我们需要根据NGINX的配置来选择正确的方式获取IP。