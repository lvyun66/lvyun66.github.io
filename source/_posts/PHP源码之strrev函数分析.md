---
title: PHP源码之strrev函数分析
date: 2019-04-09 19:03:01
tags:
- PHP
- 源码
categories:
- PHP
comments: true
---

PHP函数`strrev`的作用是反转字符串，传入一个带反转的字符串，并返回反转后的字符串。

<!-- more -->

函数原型如下：
```php
strrev(string $str): string
```

PHP官方文档：[strrev](https://www.php.net/manual/en/function.strrev.php) 描述了函数的具体使用方法。

使用场景如下：
1. 反转字符串或一串数字
2. 判断数字是否为回文数

## 函数源码实现

源文件位于`php-src/ext/standard/string.c`文件中，是PHP标准扩展string字符串扩展中的一个函数。

```c
/* {{{ proto string strrev(string str)
   Reverse a string */
#if ZEND_INTRIN_SSSE3_NATIVE
#include <tmmintrin.h>
#endif
PHP_FUNCTION(strrev)    // 声明函数名称 strrev
{
    // strrev参数str，类型为zend_string类型的指针
    zend_string *str;
    // 分别表示参数的开始字符和结束字符地址
    const char *s, *e;
    // 函数结果n的值的地址
    char *p;
    // 函数返回结果n，类型同样为zend_string类型的指针
    zend_string *n;

    // 解析函数参数并绑定到str
    ZEND_PARSE_PARAMETERS_START(1, 1)
        Z_PARAM_STR(str)
    ZEND_PARSE_PARAMETERS_END();

    // 申请长度为 ZSTR_LEN(str) 的内存，STR_LEN(str)为字符串长度
    n = zend_string_alloc(ZSTR_LEN(str), 0);
    // 返回结果n的值的地址，通过地址的方式访问字符串
    p = ZSTR_VAL(n);

    // 获取str的首元素的指针
    s = ZSTR_VAL(str);
    // 字符串的末位字符地址
    e = s + ZSTR_LEN(str);
    // 防止内存越界，保证e在s <= e < s + ZSTR_LEN(str)范围之内
    --e;
    // 这里还没理解作用是什么？
#if ZEND_INTRIN_SSSE3_NATIVE
    if (e - s > 15) {
        const __m128i map = _mm_set_epi8(
                0, 1, 2, 3,
                4, 5, 6, 7,
                8, 9, 10, 11,
                12, 13, 14, 15);
        do {
            const __m128i str = _mm_loadu_si128((__m128i *)(e - 15));
            _mm_storeu_si128((__m128i *)p, _mm_shuffle_epi8(str, map));
            p += 16;
            e -= 16;
        } while (e - s > 15);
    }
#endif
    while (e >= s) { // 终止条件为，当指向str末尾的指针小于开头的指针，终止循环
        // 把str的尾字符与n的首字符交换，达到反转效果
        *p++ = *e--;
    }
    // 将指针赋值为空
    *p = '\0';
    // 返回结果n
    RETVAL_NEW_STR(n);
}
/* }}} */
```

从源码可以看出，反转字符串用到了两个数组，一个是参数、另一个是返回结果。函数通过遍历参数str，实现函数的反转。

当字符串长度大于15可能会对函数进行优化。具体如何优化，后面对源码理解更加深刻之后再来填坑。