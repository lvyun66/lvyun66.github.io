---
title: fopen中mode详解
date: 2018-12-10 16:49:14
tags:
- PHP
- fopen
categorites:
- PHP
comments: true
---

文件操作是一个很平常的一个操作，打开文件、写入文件已经关闭文件等操作。不同模式打开文件会有不同的效果。有一些`mode`会比较容易混淆，在此记录下它们之间的区别。
<!-- more -->
`mode`模式：

1. `r`：以只读模式打开文件，将文件指针指向文件头；
2. `r+`：以读写模式打开文件，将文件指针指向开头；
3. `w`：以只读模式打开文件，将文件指针指向文件头并将文件大小截为0，如果文件不存在则尝试创建；
4. `w+`：以读写模式打开文件，将文件指正指向文件头并将文件大小截为0，如果文件不存在则尝试创建；
5. `a`：以写入模式打开文件，将文件指针指向文件结尾，如果文件不存在则尝试创建；
6. `a+`：以读写模式打开文件，将文件指针指向文件结尾，如果文件不存在则尝试创建；
7. `x`：创建并以写入模式打开文件，将文件指正指向文件开头，如果文件已存在，则`fopen`调用失败，返回`false`，如果不存在则尝试创建；
8. `x+`：创建并读写模式打开文件，将文件指针指向文件开头，如果文件已存在，则`fopen`调用失败，返回`false`，如果不存在则尝试创建；

其中我们最容易混淆的两个模式就是`r+`和`w+`，这两个模式都是以读写方式打开文件，但之间有何区别呢？

`r+`不会将文件清空，将文件指针指向文件开头，如果文件不为空，内容会被新写入的覆盖。

假设文件`test.txt`内容如下：
```text
This is testing for fprintf 5...
This is testing for fputs 6
This is testing for fprintf 7...
This is testing for fputs 8
```

php代码：
```php
<?php

$filename = 'test.txt';
$fp = fopen($filename, 'r+');
fprintf($fp, "This is testing for fprintf 7...\n");
fputs($fp, "This is testing for fputs 8\n");
fclose($fp);
```

最终结果：
```txt
This is testing for fprintf 7...
This is testing for fputs 8
This is testing for fprintf 7...
This is testing for fputs 8
```

`w+`会清空文件内容，将文件指正指向文件开头，类似与先将文件删除，再新建文件，最后把新内容写入文件中；如果要打开的文件不存在则会创建。


That`s all！Thx.