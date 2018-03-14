---
title: Golang中Timer的实现
date: 2018-2-13 17:50:46
tags:
    - Golang
    - timer
categories:
    - Golang
comments: true
---

# Timer的实现

`time`包含了`Timer`、`Ticker`的实现，作用分别为定时器和计时器。定时器配合通道`channel`使用可以达到通道超时异常处理的效果。

`Timer`结构体中包含了`runtime`中`runtimeTimer`的实现，如果要了解具体是如何实现，可以去看`runtime`库，在这里只会讲`time.Timer`的实现。

定时器结构体，包含Time类型的阻塞channel，和一个runtimeTimer：

```go
type Timer struct {
	C <-chan Time
	r runtimeTimer
}
```
其中`C`是一个`Time`类型的通道。

新建一个计时器
```go
// 新建计时器的关键代码在于函数startTimer的实现
// 在sleep.go文件中，我们只能找到方法的定义，并没站到是如何具体显示的
// 此函数可以在runtime包中找到
func NewTimer(d Duration) *Timer {
	c := make(chan Time, 1)
	t := &Timer{
		C: c,
		r: runtimeTimer{
			when: when(d),      // 计时时间，过多少时间后回调函数 sendTime()
			f:    sendTime,     // 发送Time到Timer结构体中
			arg:  c,            // 回调函数的参数
		},
	}
	// 编译过程中golang会自动将startTime()函数翻译成runtime.startTimer()
	// 将runtimeTimer结构体翻译为runtime.Timer结构
	startTimer(&t.r)            // 开始计时，关键代码在此
	return t
}

// 其中sendTime源码如下,主要作用就是发送一个时间到c.Time
// 关键：c.(chan Time) <- Now()
// 以下为翻译:
// 非阻塞发送time到c结构体
// 在NewTimer中使用,不会被阻塞
// 在NewTicker中使用
func sendTime(c interface{}, seq uintptr) {
	// Non-blocking send of time on c.
	// Used in NewTimer, it cannot block anyway (buffer).
	// Used in NewTicker, dropping sends on the floor is
	// the desired behavior when the reader gets behind,
	// because the sends are periodic.
	select {
	case c.(chan Time) <- Now():
	default:
	}
}

// 返回channel，利用通道的阻塞特性，时间到之后执行分支代码功能
func After(d Duration) <-chan Time {
	return NewTimer(d).C
}

// 关闭计时器
func (t *Timer) Stop() bool {
	if t.r.f == nil {
		panic("time: Stop called on uninitialized Timer")
	}
	return stopTimer(&t.r)
}

// 充值计时器
func (t *Timer) Reset(d Duration) bool {
	if t.r.f == nil {
		panic("time: Reset called on uninitialized Timer")
	}
	w := when(d)
	active := stopTimer(&t.r)
	t.r.when = w
	startTimer(&t.r)
	return active
}
```

## runtime包中startTimer和Timer{}的实现

[runtime包中的startTimer实现](https://github.com/golang/go/blob/871b79316ad7f2b10f1347f8d9077713afaff451/src/runtime/time.go#L106)，具体是在`runtime/time.go`文件中

```go
func startTimer(t *timer) {
	if raceenabled {
		racerelease(unsafe.Pointer(t))
	}
	addtimer(t)
}

func addtimer(t *timer) {
	tb := t.assignBucket()
	lock(&tb.lock)
	tb.addtimerLocked(t)
	unlock(&tb.lock)
}

// Add a timer to the heap and start or kick timerproc if the new timer is
// earlier than any of the others.
// Timers are locked.
func (tb *timersBucket) addtimerLocked(t *timer) {
	// when must never be negative; otherwise timerproc will overflow
	// during its delta calculation and never expire other runtime timers.
	if t.when < 0 {
		t.when = 1<<63 - 1
	}
	t.i = len(tb.t)
	tb.t = append(tb.t, t)
	siftupTimer(tb.t, t.i)
	if t.i == 0 {
		// siftup moved to top: new earliest deadline.
		if tb.sleeping {
			tb.sleeping = false
			notewakeup(&tb.waitnote)
		}
		if tb.rescheduling {
			tb.rescheduling = false
			goready(tb.gp, 0)
		}
	}
	if !tb.created {
		tb.created = true
		go timerproc(tb)
	}
}

// Timerproc runs the time-driven events.
// It sleeps until the next event in the tb heap.
// If addtimer inserts a new earlier event, it wakes timerproc early.
func timerproc(tb *timersBucket) {
	tb.gp = getg()
	for {
		lock(&tb.lock)
		tb.sleeping = false
		now := nanotime()
		delta := int64(-1)
		for {
			if len(tb.t) == 0 {
				delta = -1
				break
			}
			t := tb.t[0]
			delta = t.when - now
			if delta > 0 {
				break
			}
			if t.period > 0 {
				// leave in heap but adjust next time to fire
				t.when += t.period * (1 + -delta/t.period)
				siftdownTimer(tb.t, 0)
			} else {
				// remove from heap
				last := len(tb.t) - 1
				if last > 0 {
					tb.t[0] = tb.t[last]
					tb.t[0].i = 0
				}
				tb.t[last] = nil
				tb.t = tb.t[:last]
				if last > 0 {
					siftdownTimer(tb.t, 0)
				}
				t.i = -1 // mark as removed
			}
			f := t.f
			arg := t.arg
			seq := t.seq
			unlock(&tb.lock)
			if raceenabled {
				raceacquire(unsafe.Pointer(t))
			}
			f(arg, seq)
			lock(&tb.lock)
		}
		if delta < 0 || faketime > 0 {
			// No timers left - put goroutine to sleep.
			tb.rescheduling = true
			goparkunlock(&tb.lock, "timer goroutine (idle)", traceEvGoBlock, 1)
			continue
		}
		// At least one timer pending. Sleep until then.
		tb.sleeping = true
		tb.sleepUntil = now + delta
		noteclear(&tb.waitnote)
		unlock(&tb.lock)
		notetsleepg(&tb.waitnote, delta)
	}
}

```

`runtime/time.go`包含了`time`包中所有的基本实现过程，有兴趣可以深入学习
