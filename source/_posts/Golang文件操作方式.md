---
title: Golang文件操作方式
date: 2018-3-21 10:57:03
tags:
- Golang
- 文件
categories:
- Golang
comments: true
---

文件的操作是开发中常遇到的问题，打开文件读取内容或者是把内容写入到文件中，都会用到文件的操作；如果不熟悉文件操作，会降低开发效率。下面介绍开发中会用到的几种Go文件操作方式。
<!-- more -->
## File概念

GO中文件操作是基于file的，go会判断操作系统选择对应的`*file`，其中window与unix下的区别主要在于`unix`多一个`nonblock`，也就是非阻塞。
```go
type File struct {
    *file // os specific
}

// window
type file struct {
	pfd     poll.FD  // 文件句柄
	name    string   // 文件名称
	dirinfo *dirInfo // nil unless directory being read
}

// unix
type file struct {
	pfd      poll.FD
	name     string
	dirinfo  *dirInfo // nil unless directory being read
	nonblock bool     // whether we set nonblocking mode
}
```

## OS包

```go
// CreateByOS 使用os库创建文件
func CreateByOS() {
	file, _ := os.Create("io_test.txt")
	defer file.Close()
	file.WriteString("Created by os create.\n")
	file.Write([]byte("Write something byte to io_test.txt!\n"))

	buf := bytes.Buffer{}
	buf.Write([]byte("写入时间："))
	buf.Write([]byte(time.Now().Format("2006-01-02 15:04:05")))

	file.Write(buf.Bytes())
	file.WriteAt([]byte("这是中间插入的"), 7)
}

// ReadByOS 使用os库读取文件
func ReadByOS() {
	txt := make([]byte, 1024)
	file, err := os.Open("io_test.txt")
	if err != nil {
		panic(err)
	}
	_, err = file.Read(txt)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(txt))
}
```

## ioutil包

```go
func WriteByIOUtil() {
	ioutil.WriteFile("io_test.txt", []byte("这是用ioutil包写入的信息"), 0666)
}

func ReadByIOUtil() {
	txt, _ := ioutil.ReadFile("io_test.txt")
	fmt.Println(string(txt))

	file, _ := os.Open("io_test.txt")
	txt, _ = ioutil.ReadAll(file)
	fmt.Println(string(txt))
}
```
如果看过源码会发现，`ioutil.ReadFile()`最终是调用`ioutil.readAll()`函数；而`ioutil.ReadAll()`的底层也是调用`ioutil.readAll()`。

`ReadFile`相对于`ReadAll`只是多了一部打开文件操作，同时判断是否为文件。`ReadFile`默认的切片大小为`文件Size + bytes.MinRead512`，因此在扩展切片的时候不需要重新分配地址，速度会比`ReadAll()`快，效率高。因此读取整个文件建议使用`ReadFile`。
```go
buf := bytes.NewBuffer(make([]byte, 0, capacity))
```


而其中`readAll()`又是调用`bytes包`的`ReadFrom()`函数，可以认为`ioutil`是在`bytes`之上在封装了一层。

## bufio

```go
func WriteByBufio() {
	file, _ := os.OpenFile("io_test.txt", os.O_RDWR|os.O_APPEND, 0666)
	buf := bufio.NewWriter(file)
	buf.WriteString("这是使用bufio追加的信息！\n")
	buf.Flush()
	defer file.Close()
}

func ReadByBufio() {
	file, _ := os.Open("io_test.txt")
	buf := bufio.NewReader(file)
	txt := make([]byte, 1024)
	buf.Read(txt)
	fmt.Println(string(txt))
	defer file.Close()
}
```


```go
// Read reads data into p.
// It returns the number of bytes read into p.
// The bytes are taken from at most one Read on the underlying Reader,
// hence n may be less than len(p).
// At EOF, the count will be zero and err will be io.EOF.
func (b *Reader) Read(p []byte) (n int, err error) {
    // 切片长度
	n = len(p)
    // 如果切片长度为0，则返回0和nil
	if n == 0 {
		return 0, b.readErr()
	}
    // r w是buf读取的位置，如果r==w，表示是起点
	if b.r == b.w {
        // 开始时有错误直接抛出
		if b.err != nil {
			return 0, b.readErr()
		}
        // 如果切片长度 > 缓冲长度，直接赋值给p，避免赋值；
        // 正常情况应该先赋值给b.buf, 然后通过赋值给p，这样造成了资源的浪费
		if len(p) >= len(b.buf) {
			// Large read, empty buffer.
			// Read directly into p to avoid copy.
			n, b.err = b.rd.Read(p)
			if n < 0 {
				panic(errNegativeRead)
			}
			if n > 0 {
				b.lastByte = int(p[n-1])
				b.lastRuneSize = -1
			}
			return n, b.readErr()
		}
		// One read.
		// Do not use b.fill, which will loop.
		b.r = 0
		b.w = 0
		n, b.err = b.rd.Read(b.buf)
		if n < 0 {
			panic(errNegativeRead)
		}
		if n == 0 {
			return 0, b.readErr()
		}
		b.w += n
	}

	// copy as much as we can
	n = copy(p, b.buf[b.r:b.w])
	b.r += n
	b.lastByte = int(b.buf[b.r-1])
	b.lastRuneSize = -1
	return n, nil
}
```
