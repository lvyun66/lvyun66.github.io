---
title: Golang循环队列实现
date: 2018-09-09 10:55:46
tags:
    - Golang
    - Queue
categories:
    - Golang
comments: true
---


## LeetCode循环队列实现

最初的实现方法:
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