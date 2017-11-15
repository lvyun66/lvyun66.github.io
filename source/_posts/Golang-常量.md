---
title: Golang
date: 2017/11/15 上午9:30:46
tags:
    - Golang
    - const
categories:
    - Golang
comments: true
---

# Golang常量

## 1. 定义
`关键字` `常量名` = `常量值`
```go
const Pi float32 = 3.14    // 显示类型定义
const Pi = 3.14            // 隐式类型定义
```

如果隐式类型定义,编译器会自动根据值类型判断常量类型

## 2. iota

`iota`可以在常量定义中自增, 例如:
```go
const (
    Sunday = iota           // 0
    Monday                  // 1
    Tuesday                 // 2
    Wednesday               // 3
    Thursday                // ...
    Friday
    Saturday
)
```

又例如:

```go
const (
    A, B = iota + 1, iota + 2
    C, D
    E, F
)
```
最后的结果为:
```
A = 1
B = 2
C = 2
D = 3
E = 3
F = 4
```
因为`iota`只会在下一行才会自增; 在本行定义中会始终保持不变

`iota`在没遇到一个常量快或者单个常量声明时, `iota`都会置为0; 也就是说`iota`在每次遇到关键字`const`时, 都会重置为`0`;

## 3. 自定义类型

```go
type Color int

const (
    RED Color = iota // 0
    ORANGE // 1
    YELLOW // 2
    GREEN // ..
    BLUE
    INDIGO
    VIOLET // 6
)
```

如果函数参数为`Color`类型, 而传入参数类型为`int`, 则会在编译过程中报错;

```
# command-line-arguments
.\main.go:45:11: cannot use color (type Color) as type int in argument to EchoColor
```

## 4. 表达式
常量定义中可以包含表达式, 但是不能包含函数或者方法; 因为函数和方法在编译过程中是未知的; 例如在[Effective.go](https://golang.org/doc/effective_go.html#constants)文件中:
```go
type ByteSize float64

const (
    _           = iota                   // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota) // 1 << (10*1)
    MB                                   // 1 << (10*2)
    GB                                   // 1 << (10*3)
    TB                                   // 1 << (10*4)
    PB                                   // 1 << (10*5)
    EB                                   // 1 << (10*6)
    ZB                                   // 1 << (10*7)
    YB                                   // 1 << (10*8)
)
```

如果我们想忽略某个常量, 可以使用`_`空白字符来代替;

## 附录
`<<`位运算符
