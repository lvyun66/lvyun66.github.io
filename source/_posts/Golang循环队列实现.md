---
title: Golang循环队列实现
date: 2018-09-09 10:55:46
tags:
    - Golang
    - Queue
    - CircularQueue
categories:
    - Golang
comments: true
---

本文主要是对`leetcode`中队列的实现做一个简单的记录和一些心得.
<!-- more -->
## LeetCode循环队列实现

### 最初的实现方法
实现思路: 实现的方法很简单,只需要注意如何判断队列是否为空或者已经满;
而struct中的size字段则是用于判断队列是否为空或者队列已满的,需要利用`size`字段来实现这些功能.

代码如下:
```golang
type MyCircularQueue struct {
    data   []int
    maxCap int
    size   int
    head   int
    tail   int
}


/** Initialize your data structure here. Set the size of the queue to be k. */
func Constructor(k int) MyCircularQueue {
    d := make([]int, k)
    return MyCircularQueue{
        data:   d,
        maxCap: k,
        size:   0,
        head:   -1,
        tail:   -1,
    }
}


/** Insert an element into the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) EnQueue(value int) bool {
    if this.IsFull() {
        return false
    }
    if (this.IsEmpty()) {
        this.head = 0
        this.tail = 0
    } else {
        this.tail ++
    }
    if this.tail >= this.maxCap {
        this.tail = 0
    }
    this.data[this.tail] = value
    this.size++

    return true
}


/** Delete an element from the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) DeQueue() bool {
    if this.IsEmpty() {
        return false
    }
    if this.head >= this.maxCap {
        this.head = 0
    }
    _ = this.data[this.head]
    this.head++
    this.size--
    if this.size == 0 {
        this.head = -1
        this.tail = -1
    }

    return true
}


/** Get the front item from the queue. */
func (this *MyCircularQueue) Front() int {
    if this.IsEmpty() {
        return -1
    }
    return this.data[this.head]
}


/** Get the last item from the queue. */
func (this *MyCircularQueue) Rear() int {
    if this.IsEmpty() {
        return -1
    }
    return this.data[this.tail]
}


/** Checks whether the circular queue is empty or not. */
func (this *MyCircularQueue) IsEmpty() bool {
    return this.size == 0
}


/** Checks whether the circular queue is full or not. */
func (this *MyCircularQueue) IsFull() bool {
    return this.size == this.maxCap
}


/**
 * Your MyCircularQueue object will be instantiated and called as such:
 * obj := Constructor(k);
 * param_1 := obj.EnQueue(value);
 * param_2 := obj.DeQueue();
 * param_3 := obj.Front();
 * param_4 := obj.Rear();
 * param_5 := obj.IsEmpty();
 * param_6 := obj.IsFull();
 */
```

后续优化方法会在后面更新出来, 敬请期待...

### 2018-09-09 23:17:44 Sunday 更新.

第二种思路:

思路：和第一种解法不同在于，struct减少maxCap字段，同是用size表示队列的容量

难题1：如何判断队列是否为空？
```text
判断tail==-1，在第一个元素入队时，会将tail、head同是设置为0，指向第一个元素；
队列只有最后一个元素时，讲head、tail置为-1；这样可以保证队列为空时head、tail为-1
```

难题2：如何判断队列已满？
```text
只有队列中包含一个元素或者为空时，head==tail，其他情况head时不可能等于tail
当队列已满时，如果再有一个元素需要入队，在tail==head，因此可以用这个零界点判断队列是否已满
(this.tail+1)%this.size == this.head
```

难题3：队列为空，元素入队时如何处理head、tail的值？
```text
队列为空，有元素如队列，需要同时将head、tail置为0；同时用(this.tail + 1) % this.size保证可循环
```

难题4：队列只有一个元素，需要出队时，如果处理tail、head？
```text
队列只有一个元素，当需要出队列时。同时将tail、head置为-1；如果判断队列之后一个元素，只需要判断head==tail即可
```

代码如下:
```golang
// MyCircularQueue
type MyCircularQueue struct {
    size, head, tail int
    elements         []int
}

/** Initialize your data structure here. Set the size of the queue to be k. */
func Constructor(k int) MyCircularQueue {
    return MyCircularQueue{
        size:     k,
        head:     -1,
        tail:     -1,
        elements: make([]int, k),
    }
}

/** Insert an element into the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) EnQueue(value int) bool {
    if this.IsFull() {
        return false
    }
    if this.tail == -1 {
        this.head = 0
    }
    this.tail = (this.tail + 1) % this.size
    this.elements[this.tail] = value
    return true
}

/** Delete an element from the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) DeQueue() bool {
    if this.IsEmpty() {
        return false
    }
    _ = this.elements[this.head]
    if this.head == this.tail {
        this.head = -1
        this.tail = -1
    } else {
        this.head = (this.head + 1) % this.size
    }
    return true
}

/** Get the front item from the queue. */
func (this *MyCircularQueue) Front() int {
    if this.IsEmpty() {
        return -1
    }
    return this.elements[this.head]
}

/** Get the last item from the queue. */
func (this *MyCircularQueue) Rear() int {
    if this.IsEmpty() {
        return -1
    }
    return this.elements[this.tail]
}

/** Checks whether the circular queue is empty or not. */
func (this *MyCircularQueue) IsEmpty() bool {
    return this.tail == -1
}

/** Checks whether the circular queue is full or not. */
func (this *MyCircularQueue) IsFull() bool {
    return (this.tail+1)%this.size == this.head
}

/**
 * Your MyCircularQueue object will be instantiated and called as such:
 * obj := Constructor(k);
 * param_1 := obj.EnQueue(value);
 * param_2 := obj.DeQueue();
 * param_3 := obj.Front();
 * param_4 := obj.Rear();
 * param_5 := obj.IsEmpty();
 * param_6 := obj.IsFull();
 */
```

###  2018-09-11 23:49:44 Tuesday 更新

第三种思路: 与第二种思路类似,不同地方在于`elements`的长度.
`elements`的长度设置为`队列长度+1`, 这样做的目的在于, 当队列为空时,不用重复讲`head`和`tail`置为-1, 可以循环入队出队.

难点1: 如果判断队列是否为空?
```text
在最开始, head和tail的值都为0, 而无论队列如何操作, 当队列为空的时候head永远是等于tail.
例如: 队列的大小为5, 连续5次入队后,tail的值为4,而head值为0;
连续五次出队, tail值为4, 而head值同样等于4, 此时队列为空.
因此head==tail时,队列为空.
```

难点2: 如何判断队列已满?
```text
因为队列的实际大小比要求的队列大小大1, 因此当(tail+1)%队列实际大小 == head时, 队列已满.
```

代码如下:
```golang
type MyCircularQueue struct {
    size, head, tail int
    elements         []int
}

/** Initialize your data structure here. Set the size of the queue to be k. */
func Constructor(k int) MyCircularQueue {
    return MyCircularQueue{
        size:     k + 1,
        head:     0,
        tail:     0,
        elements: make([]int, k+1),
    }
}

/** Insert an element into the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) EnQueue(value int) bool {
    if this.IsFull() {
        return false
    }
    this.tail = (this.tail + 1) % this.size
    this.elements[this.tail] = value
    return true
}

/** Delete an element from the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) DeQueue() bool {
    if this.IsEmpty() {
        return false
    }
    this.head = (this.head + 1) % this.size
    _ = this.elements[this.head]
    return true
}

/** Get the front item from the queue. */
func (this *MyCircularQueue) Front() int {
    if this.IsEmpty() {
        return -1
    }
    return this.elements[this.head+1]
}

/** Get the last item from the queue. */
func (this *MyCircularQueue) Rear() int {
    if this.IsEmpty() {
        return -1
    }
    return this.elements[this.tail]
}

/** Checks whether the circular queue is empty or not. */
func (this *MyCircularQueue) IsEmpty() bool {
    return this.head == this.tail
}

/** Checks whether the circular queue is full or not. */
func (this *MyCircularQueue) IsFull() bool {
    return (this.tail+1)%this.size == this.head
}

/**
 * Your MyCircularQueue object will be instantiated and called as such:
 * obj := Constructor(k);
 * param_1 := obj.EnQueue(value);
 * param_2 := obj.DeQueue();
 * param_3 := obj.Front();
 * param_4 := obj.Rear();
 * param_5 := obj.IsEmpty();
 * param_6 := obj.IsFull();
 */

```

### 终结
目前, 只能想到这三种办法实现循环队列, 如果大家有更好的想法,可以一起沟通.

### See you next time