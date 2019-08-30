---
title: 优雅的关闭Golang的Channel
date: 2018-02-20 17:50:46
tags:
- Golang
- channel
categories:
- Golang
comments: true
---

几天前，我写过一篇解释`Golang的Channel规则`的文章。这篇文章获得很多的点赞在`reddit`和`HN`上。同时也收集了一些关于`Go channel的设计和规则`的评论：
1. 没有一种简单通用的方式去检查通道是否已经关闭，在没有修改通道状态的情况下。
2. 关闭一个已经关闭的channel会panic，如果在阻塞者在不知道通道是否已经关闭的情况下尝试关闭通道是是危险的。
3. 发送数据给一个已经关闭的通道会panic，如果在发送者在不知道通道是否已经关闭的情况下发送数据给通道是危险的。

<!-- more -->

评论看上去是合理的（实际上并不是）。是的，没有一个内置函数去检测通道是否被关闭。

如果你可以确保没有值被发送至通道，确实有一个简单的方法检测通道是否被关闭（这个方法会在文章中常用到）：
```go
package main

import "fmt"

type T int

func IsClosed(ch <-chan T) bool {
	select {
	case <-ch:
		return true
	default:
	}

	return false
}

func main() {
	c := make(chan T)
	fmt.Println(IsClosed(c))
	close(c)
	fmt.Println(IsClose(c))
}

```


## 已经关闭的`channel`不会阻塞
## 一个`nil channel`永远是阻塞的
