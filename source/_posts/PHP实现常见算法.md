---
title: PHP实现常见算法
date: 2018-10-25 09:49:15
tags:
- PHP
- 算法
categories:
- 算法
comments: true
---

PHP版排序算法的实现，包括思路、语言实现以及部分算法的优化。算法主要包括冒泡排序、插入排序、选择排序、归并排序、快速排序、希尔排序和桶排序、计数排序以及基数排序等。其中最后三种虽然不常见，但是他们的思想在很多地方都能用到。

<!-- more -->

## 冒泡排序
```php
function bubble_sort(array $arr)
{
    if (count($arr) <= 1) {
        return $arr;
    }
    for ($i = 0; $i < count($arr); $i++) {
        for ($j = $i; $j < count($arr); $j++) {
            if ($arr[$i] > $arr[$j]) {
                $tmp = $arr[$i];
                $arr[$i] = $arr[$j];
                $arr[$j] = $tmp;
            }
        }
    }

    return $arr;
}
```

### 优化

## 插入排序
```php
function insert_sort(array $arr)
{
    if (count($arr) <= 1) {
        return $arr;
    }
    for ($i = 0; $i < count($arr); $i++) {
        for ($j = 0; $j <= $i; $j++) {
            if ($arr[$i] < $arr[$j]) {
                $tmp = $arr[$j];
                $arr[$j] = $arr[$i];
                $arr[$i] = $tmp;
            }
        }
    }

    return $arr;
}
```

## 选择排序
```php
function selection_sort(array $arr)
{
    if (count($arr) <= 1) {
        return $arr;
    }
    for ($i = 0; $i < count($arr); $i++) {
        $min = $i;
        for ($j = $i; $j < count($arr); $j++) {
            if ($arr[$min] > $arr[$j]) {
                $min = $j;
            }
        }
        $tmp = $arr[$i];
        $arr[$i] = $arr[$min];
        $arr[$min] = $tmp;
    }

    return $arr;
}
```

## 归并排序
```php
function merge_sort(array $arr)
{
    $len = count($arr);
    if ($len <= 1) {
        return $arr;
    }

    $mid = (int)floor($len / 2);
    $arr = array_merge(merge_sort(array_slice($arr, 0, $mid)), merge_sort(array_slice($arr, $mid)));

    $tmp = [];
    $left = 0;
    $right = $mid;
    while ($left < $mid && $right <= $len - 1) {
        if ($arr[$left] < $arr[$right]) {
            $tmp[] = $arr[$left++];
        } else {
            $tmp[] = $arr[$right++];
        }
    }

    $offset = $right;
    $length = $len - $right;
    if ($right == $len) {
        $offset = $left;
        $length = $mid - $left;
    }
    return array_merge($tmp, array_slice($arr, $offset, $length));
}
```

## 快速排序
```php
function quick_sort(array $arr)
{
    $len = count($arr);
    if ($len <= 1) {
        return $arr;
    }
    $pivot = end($arr);
    $pivotIndex = key($arr);
    reset($arr);
    $left = key($arr);
    for ($i = 0; $i < $len - 1; $i++) {
        if ($arr[$i] < $pivot) {
            $value = $arr[$i];
            $arr[$i] = $arr[$left];
            $arr[$left] = $value;
            $left++;
        }
    }
    list($arr[$left], $arr[$pivotIndex]) = [$arr[$pivotIndex], $arr[$left]];

    return array_merge(quick_sort(array_slice($arr, 0, $left)), [$pivot], quick_sort(array_slice($arr, $left + 1)));
}
```

## 桶排序
```php
function bucket_sort(array $arr)
{
    if (count($arr) <= 1) {
        return $arr;
    }
    $mean = max($arr) / 100;
    // default 100 bucket
    $bucket = [];
    for ($i = 0; $i < 100; $i++) {
        $bucket[$i] = [];
    }
    foreach ($arr as $value) {
        $bucket[(int)floor($value / $mean)][] = $value;
    }
    foreach ($bucket as $key => $item) {
        $bucket[$key] = quick_sort($item);
    }

    return array_merge(...$bucket);
}
```

## 计数排序
```php
function counting_sort(array $arr)
{
    if (count($arr) <= 1) {
        return $arr;
    }
    $counting = [];
    $max = max($arr);
    for ($i = 0; $i <= $max; $i++) {
        $counting[$i] = 0;
    }
    foreach ($arr as $value) {
        $counting[$value] += 1;
    }
    for ($i = 1; $i <= $max; $i++) {
        $counting[$i] += $counting[$i - 1];
    }
    $tmp = [];
    for ($i = count($arr) - 1; $i >= 0; $i--) {
        $index = $counting[$arr[$i]] - 1;
        $tmp[$index] = $arr[$i];
        $counting[$arr[$i]]--;
    }
    foreach ($tmp as $key => $value) {
        $arr[$key] = $tmp[$key];
    }
    return $arr;
}
```

## 基数排序
```php
function radix_sort()
{

}
```
