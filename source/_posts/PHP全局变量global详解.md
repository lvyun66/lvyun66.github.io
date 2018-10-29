---
title: PHP全局变量global详解
date: 2018-10-16 14:36:07
tags:
    - PHP
categories:
    - PHP
comments: true
---
今天在阅读一个项目源码时候碰到了PHP的`global`关键词和`$GLOBALS`超全局变量的使用，看到这两个关键词的时候有点懵逼，虽然知道这是干嘛的，但是对其使用方式不是很清楚。特在此记录下学习的过程。

## 变量范围

变量的范围即它定义的上下文背景（也就是它的生效范围）。具体信息见[PHP文档-变量](http://php.net/manual/zh/language.variables.scope.php)

在用户自定义函数中，一个局部函数范围将被引入。

**任何用于函数内部的变量按缺省情况将被限制在局部函数范围内。**

也就是说在函数中无法直接使用全局变量，如果在函数内用到全局变量但是却没有声明，PHP会提示变量未定义。
```php
$name = 'lvyun';

function say() {
    echo "I am {$name}";
}
say();

// output:
// PHP Notice:  Undefined variable: name in /data/php/chat/awesome on line 5
// I am %
```

PHP与C和C++语言有点不相同，C语言中全局变量在函数中自动生效，不需要通过关键词来引用。

了解变量的作用范围，才能理解全局变量。

## 声明全局变量

使用global关键词
```php
$name = 'lvyun';

function say() {
    global $name;
    echo "I am {$name}";
}
say();

// output
// I am lvyun%
```

当然我们也可以使用`$GLOBALS`来代替`global`, `$GLOBALS`是超全局变量, 是一个关联数组:
```php
$name = 'lvyun';

function say() {
    echo "I am {$GLOBALS['name']}";
}
say();

// output
// I am lvyun%
```

如果我们没有在函数外预先声明变量`$name`，而在函数里面直接使用变量会如何呢？

```php
function say() {
    global $name;
    echo "I am {$name}";
}
say();

// output
// I am %
```

如果我们在函数内声明全局变量，并且对变量赋值，在函数之外引用变量会如何？

```php
function say() {
    global  $name;
    echo "I am {$name}\n";
    $name = 'lvyun-inner';
}
say();
echo "I am {$name}\n";
// output
// I am
// I am lvyun-inner
```

在函数外对函数内声明的全局变量重新赋值会如何？
```php
function say() {
    global  $name;
    echo "I am {$name}\n";
    $name = 'lvyun-inner';
}
say();
echo "I am {$name}\n";
$name = 'lvyun-outer';
echo "I am {$name}\n";

// output
// I am
// I am lvyun-inner
// I am lvyun-outer
```

**在函数中声明全局变量后，对变量的所有引用都会指向其全部版本。**

## 释放全局变量

`global`和`$BLOBALS`用`unset`函数释放变量会发生什么？

```php
$name = 'lvyun';
function change()
{
    global $name;
    echo "(1)I am {$name}\n";
    unset($name);
    echo "(2)I am {$name}\n";
}

change();
echo "(3)I am {$name}\n";

// output
// (1)I am lvyun
// PHP Notice:  Undefined variable: name in /data/php/chat/awesome on line 49
// (2)I am
// (3)I am lvyun
```

在`unset`之后，函数内的全局变量`$name`无法使用，而且函数外的`$name`任然存在。

```php
$name = 'lvyun';
function change()
{
    echo "(1)I am {$GLOBALS['name']}\n";
    unset($GLOBALS['name']);
    echo "(2)I am {$GLOBALS['name']}\n";
}

change();
echo "(3)I am {$name}\n";

// output
// (1)I am lvyun
// PHP Notice:  Undefined index: name in /data/php/chat/awesome on line 61
// (2)I am
// PHP Notice:  Undefined variable: name in /data/php/chat/awesome on line 65
// (3)I am
```

在`unset`之后，函数内外的全局变量`$name`都无法使用。

```php
$name = 'lvyun';
function change()
{
    global $name;
    echo "(1)I am {$name}\n";
    echo "(2)I am {$GLOBALS['name']}\n";
    unset($$GLOBALS['name']);
    echo "(3)I am {$name}\n";
    echo "(4)I am {$GLOBALS['name']}\n";
}

change();
echo "(5)I am {$name}\n";

// output
// (1)I am lvyun
// (2)I am lvyun
// (3)I am lvyun
// PHP Notice:  Undefined index: name in /data/php/chat/awesome on line 75
// (4)I am
// PHP Notice:  Undefined variable: name in /data/php/chat/awesome on line 79
// (5)I am
```

在`unset`后，函数内的变量`$name`任然可以使用，而超全局变量`$GLOBALS['name']`却不能使用。

通过上面说明测试说明：
```php
global $var 是申明一个局部函数变量，函数中的 $var 和 全局 $var 具有相同引用。

$GLOBALS['var'] 则指的是变量本身。

global $var 等价于 $var = &$GLOBALS['var']
```

这里引用鸟哥在 [PHP源码分析之Global关键字](http://www.laruence.com/2008/08/24/377.html) 的一句话：

**如果你global了一个变量，那么Zend就会去全局symbol_table去寻找，如果找不到，就会在全局symbol_table中分配相应的变量。**
