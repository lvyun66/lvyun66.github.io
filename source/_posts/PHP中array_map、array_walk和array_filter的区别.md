---
title: PHP数组函数
date: 2018-3-23 10:39:20
tags:
    - PHP
    - array_map
    - array_walk
    - array_filter
categories:
    - PHP
comments: true
---

PHP数组是开发中最常用的数据类型，同时PHP也内置很多关于数组的扩展。这里主要介绍`array_map`、`array_walk`和`array_filter`三者的区别和应用场景，因为这三者很容易混淆。
<!-- more -->
这三个函数都用到了回调函数`callback`，都作用于数组的每个单元之上。其中最容易混淆的就是array_map和array_walk。

三者在PHP官方文档的链接分别如下，建议仔细阅读官方文档：
[array_map](http://php.net/manual/zh/function.array-map.php)
[array_walk](http://php.net/manual/zh/function.array-walk.php)
[array_filter](http://php.net/manual/zh/function.array-filter.php)


## array_map

返回数组，是为 array1 每个元素应用 callback函数之后的数组。 callback 函数形参的数量和传给 array_map() 数组数量，两者必须一样。
```php
array_map($callback, $array1 [, $array2])
return $newArray
```

`callback`的参数数量要是`array`保持一致。

返回一个新的数组，数组的值是`$array1`每个元素应用`回调函数callback`之后的值。*如果返回的新数组的值不为空，`callback`必须要有返回值。*

用`foreach`举例：
```php
$arr = [1, 2, 3, 4, 5];
$newArr = [];
foreach($arr as $key => $val) {
    $newArr[] = aquare($val)
}
var_dump($newArr);

function aquare($val) {
    return pow($val, 2);
}

// result:
Array
(
    [0] => 1
    [1] => 4
    [2] => 9
    [3] => 16
    [4] => 25
)

// 如果callback没有返回值。则新数组为
Array
(
    [0] =>
    [1] =>
    [2] =>
    [3] =>
    [4] =>
)
```

## array_walk

将用户自定义函数 funcname 应用到 array 数组中的每个单元。

```php
array_walk(&$array, $callback)
return bool
```

array_walk不会返回新数组，返回一个bool类型的值。典型情况下 callback 接受两个参数。array 参数的值作为第一个，键名作为第二个。

只有 array 的值才可以被改变，用户不应在回调函数中改变该数组本身的结构。例如增加/删除单元，unset 单元等等。
`array_walk`的第一个参数为引用类型，如果改变单元内容，数组内容也会改变。

```php
$arr = [
    'name' => 'lvyun',
    'age' => '23',
    'email' => 'yunlv.go@gmail.com',
];

array_walk($arr, function (&$val, $key) {
    $val = $val . " : " . time();
});

var_dump($arr);

// result
Array
(
    [name] => lvyun : 1521777271
    [age] => 23 : 1521777271
    [email] => yunlv.go@gmail.com : 1521777271
)
// 如果$val不是引用，数组就不会有任何改变
```

## array_filter

依次将 array 数组中的每个值传递到 callback 函数。如果 callback 函数返回 true，则 array 数组的当前值会被包含在返回的结果数组中。数组的键名保留不变。
```php
array_filter($arr, $callback)
return $newArr
```

实例：
```php
$arr = [
    0 => [
        'a' => 0
    ],
    1 => [
        'a' => 1
    ],
    2 => [
        'a' => 2
    ],
];

$newArr = array_filter($arr, function($item) {
    return $item['a'] == 0;
});

// result
Array
(
    [0] => Array
        (
            [a] => 0
        )
)
```

`callback`函数要返回bool类型，判断是否符合要求，如果返回true，则把该数组填入新数组中。
