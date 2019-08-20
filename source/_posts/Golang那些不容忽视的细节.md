---
title: Golang那些不容忽视的细节
date: 2019-03-20 14:32:23
tags:
    - golang
categories:
    - golang
conments: true
---

go语言的语法简单，但使用起来确有很多细节，下面是日常开发中遇到的细节问题。
<!-- more -->
## 除法运算符中关于常量的坑

两个数相除，是向下取整或向上取整还是返回浮点数？

首先，go中整数不能除以浮点数，浮点数也不能除整数，否则会发生精度值溢出的情况，这在go中是不被允许的。

其次，除法运算符的运算结果类型是由除数决定的，因为运算是从左到右的顺序执行。如果除数为整数，则结果向下取整；若为浮点数，则结果同样为浮点数。

但如果除数或者被除数都常量，结果又如何呢？我们要知道，常量可以通过常量声明或者显式的给定类型，也可以通过在变量声明、赋值或者表达式的操作数中隐式地使用；因此如果除数类型确定，而被除数类型未确定，则被除数的类型根据出来来推断。例如：
```golang
var a = 1
var b = 2
var c = 1.0
var d = 2.2
fmt.Printf("var a type = %T\n", a)
fmt.Printf("var b type = %T\n", b)
fmt.Printf("var c type = %T\n", c)
fmt.Printf("var d type = %T\n", d)
fmt.Println()

fmt.Println("a/b =", a/b)
//fmt.Println(a / d)
fmt.Println("a/2 =", a/2)
fmt.Println("a/2.0 =", a/2.0)

fmt.Println("c/d =", c/d)
fmt.Println("c/2 =", c/2)
//fmt.Println(c / b)
fmt.Println("c/2.0 =", c/2.0)

fmt.Println("1/b =", 1/b)
fmt.Println("1/2 =", 1/2)
fmt.Println("1.0/b =", 1.0/b)
fmt.Println("1.0/2,0 =", 1.0/2.0)

// output
// var a type = int
// var b type = int
// var c type = float64
// var d type = float64
//
// a/b = 0
// a/2 = 0
// a/2.0 = 0
// c/d = 0.45454545454545453
// c/2 = 0.5
// c/2.0 = 0.5
// 1/b = 0
// 1/2 = 0
// 1.0/b = 0
// 1.0/2,0 = 0.5
```

从上面的测试结果可以看出：
1. 除数为常量、被除数为整数，则结果为整数
2. 除数为整数、被除数为常量，则结果为整数
3. 除数为常量、被除数为浮点数，则结果为浮点数
4. 除数为浮点数、被除数为常量，则结果为浮点数
5. 除数为整数、被除数为浮点数。不被允许
6. 除数为浮点数、被除数为整数，不被允许

**最终我们得出一个结论，就是表达式中的常量的类型会自动根据表达式中其他的变量类型决定，如果表达式中都为常量，这返回类型为常量的默认类型**

这个关于golang中常量的类型装换的解释：[https://golang.org/ref/spec#Constants](https://golang.org/ref/spec#Constants)

## 类型转换