---
title: Git基本用户
date: 2018-3-15 20:15:04
tags:
- Git
- config
categories:
- Git
comments: true
---

本文会介绍`git`常用命令基本用法、以及常用参数；有一些是我们平常用到的，也有一些平常没有用到，但是确实很有用的参数
<!-- more -->
## log命令
`git log`是查看版本提交历史，查看历史版本中修改或者新增的内容，以及是在哪个版本修复了什么bug。
```
20:19:01 # git log
commit ddfef8044ca4fd3f2757e5f1875614ceff108424
Author: lvyun <yunlv.go@gmail.com>
Date:   Wed Mar 14 14:21:07 2018 +0800

    Update
```

常用参数：
1. `-p`
```

20:26:29 # git log -p -1 --word-diff
commit ddfef8044ca4fd3f2757e5f1875614ceff108424
Author: lvyun <yunlv.go@gmail.com>
Date:   Wed Mar 14 14:21:07 2018 +0800

    Update

diff --git "a/source/_posts/Golang\344\270\255Timer\347\232\204\345\256\236\347\216\260.md" "b/source/_posts/Golang\344\270\255Timer\347\232\204\345\256\236\347\216\260.md"
index 2d5e96b..274d8b3 100644
--- "a/source/_posts/Golang\344\270\255Timer\347\232\204\345\256\236\347\216\260.md"
+++ "b/source/_posts/Golang\344\270\255Timer\347\232\204\345\256\236\347\216\260.md"
@@ -9,7 +9,9 @@ categories:
comments: true
---

# [-time包中Timer的实现-]{+Timer的实现^M+}
{+^M+}
{+## 源代码+}

```go
// 定时器结构体，包含Time类型的阻塞channel，和一个runtimeTimer
@@ -82,7 +84,7 @@ func (t *Timer) Reset(d Duration) bool {
}
```

```
