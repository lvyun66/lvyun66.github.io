---
title: Signature数字签名
date: 2018-09-19 14:40:54
tags:
- Signature
categories:
- 数字签名
comments: true
---

签名主要使用于系统间的交互，保证接口的安全性。数字签名可以理解为之后信息发送者才能生产而别人无法伪造的一段数字串。

<!-- more -->

## Signature作用
1. 验证数据安全性（防止数据篡改）
2. 识别调用者是否真实

## Signature实现
要使`Signature`达到验证数据是否被修改的效果，生成`Signature`的过程中要加入要传输的参数，达到验证的效果。
下面是一个比较合理的签名过程：
```text
1. 对参数的Key进行排序后（ASCII排序）把参数拼接成字符串, 对字符串进行MD5加密
2. 将MD5加密后的字符串与token拼接成新字符串后在进行加密，从而得到一个Signature
3. 服务端接收到signature后对其进行验证
```

下面是用PHP实现的`Signature`生成方式：
```php
<?php

namespace yesdat\util;

class Signature
{
    public static function gen(array $params, $token, $key)
    {
        ksort($params);
        $sign = hash_hmac('sha256', md5(http_build_query($params)) . $token, $key);

        return $sign;
    }
}
```

调用如下：
```php
$key = '010203040506070809';
$token = 'f787ffb2758f5f294b4b7946f10a3ff1005f5955';
$params = [
    'q' => '{"a": 1, "b": "abc"}',
    'account_id' => 10000,
    'user_id' => 10000,
    'access_token' => $token,
];
$sign = Signature::gen($params, $token, $key);
var_dump($sign);
$this->assertTrue(true);

// 输出结果:
// e9d941d518fc5f3f98b4b386013a8a5913af246690bce5b8298cd7d3c0690a9f
```

在最后生成sign使用hmac_sha1加密方式是为了增加破解难度。

## 如何防止HTTP数据篡改？
1. 在没有http的情况下，增加`Signature`参数，可以很大程度上增加数据篡改的难度
2. 有HTTPS？请直接上HTTPS

## 重放攻击
什么是重放攻击？ 根据 [wikipedia](https://zh.wikipedia.org/wiki/%E9%87%8D%E6%94%BE%E6%94%BB%E5%87%BB) 上的记录：
```text
重放攻击（Replay attack，或称作重送攻击）是一种网络攻击，通过恶意的欺诈性地重复或拖延正常的数据传输而实施。
因工作原理如同重放歌曲一样而得名。
```

这里引用`wiki`上的图片，![重放攻击](https://upload.wikimedia.org/wikipedia/commons/c/ca/Replay_attack_on_hash.svg)

举个例子：
```text
1. 假设A在浏览器上登录C系统，但发送的请求Request1被B监听；
2. A登录并且通过检验，在系统返回成功状态；
3. 之后B可以在任何的地方重发Request1这个请求，从而获取不需要知道A的密码也能登录到系统C
```

黑客重复的发送合法请求，从而达到欺骗系统的目的！这种重复利用合法请求进行攻击称为重放。

因此防止重放就称为一件很重要的事？（废话，不防止重放攻击，账户里的钱都没了）

### 如何防止重放攻击？

防止重放攻击无非就是对系统进行`唯一性检验`。杜绝重复发送请求的可能性。

1. 时间戳(timestamp)

    时间戳为什么可以防止重放？

    一般一个HTTP请求从发出到到达服务器的时间一般不会超过60s，假设当前时间为1（timestamp=1），到达服务器进行检验时，`now_time - timestamp > 60`则认为请求是无效的。
    这样做可以杜绝大部分的攻击，但是如果黑客在60s内发起请求，我们好像还是一样的无能为力。

2. nonce

    加入一个随机数`nonce`，每次请求都会生成一个新的nonce，如果nonce已经被使用，则认为这个请求是无效的。

    生成`nonce`的方式有两种，一种是有服务端下发随机数`nonce`，发送请求时带上nonce，服务端进行检验时判断nonce是否存在。
    另一种是客户端生成，服务端记录`nonce`。

    这样做也还是会有问题：`nonce`在服务端占用的内存或者数据库越来越大，因此我们会定期清理`nonce`，但已经被清理的`nonce`就无法达到验证的效果。
    基于上面两个的缺点，产生了第三种方式。

3. timestamp+nonce

    服务端先验证时间戳是否在合法范围内。如果在，则继续验证`nonce`是否存在，存在则拒绝请求。这种方式只用存储一定时间内的`nonce`即可。

### PHP实现服务端验证过程
综上所述，用PHP实现服务端的`Signature`验证过程：
```php
$timestamp = $params['timestamp'];
if (time() - $timestamp > 60) {
    throw new \InvalidArgumentException('无效请求', 50000);
}
if (Signature::gen($params, $token, $key) !== $sign) {
    throw new \InvalidArgumentException('无效签名', 50000);
}
// nonce验证暂不验证
```
