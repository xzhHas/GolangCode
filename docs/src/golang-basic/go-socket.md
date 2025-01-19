---
title: go并发编程以及socket通信的理解
shortTitle: 19.写一个go-socket的实现
description: go并发编程以及socket通信的理解
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-05-10
---

## 一、管道的简单使用

:::tip

" golang不是通过共享内存来通信，而是通过通信来共享内存 "  

:::

1、go简单初始化

```go
// golang不是通过共享内存来通信，而是通过通信来共享内存
func a1() {
	// 声明初始化 channel
	var ch chan string = make(chan string) // deadlock 会造成死锁，因为我们的管道是没有缓冲的
	// 内置的make函数有什么作用？
	// make 初始化内存，并且返回引用类型本身
	// new 只是将内存清零，返回的是指向类型的指针
	ch <- "hello" //阻塞写
	str := <-ch   //阻塞读
	fmt.Println(str)
	// 单向channel
	var ch1 chan<- string // 只能写
	var ch2 <-chan string // 只能读
	// 关闭channel
	close(ch1)
	x, ok := <-ch2
	if ok {
		fmt.Println(x)
	} else {
		fmt.Println("channel is closed")
	}
}
```
2、用 select 做一个简单的超时管理

```go
	// Go语言直接引入select关键字，用于处理异步问题
	var ch1, ch2 chan string
	select {
	case x := <-ch1: // 如果从ch1读取数据，那么执行此语句
		fmt.Println(x)
	case y := <-ch2: // 如果从ch2读取数据，那么执行此语句
		fmt.Println(y)
	default:
		fmt.Println("default")
	}
```

超时管理：

```go
func download(ch chan string) {
	for i := 1; i < 10; i++ {
		fmt.Println(i)
		time.Sleep(time.Second * 1)
	}
	ch <- "ok"
}
func a2() {
	// 超时处理
	timeout := make(chan int, 1)
	go func() {
		time.Sleep(time.Second * 3)
		timeout <- 1 // 用来标记超时，可以是任何非0值
	}()
	ch := make(chan string, 6) // 用于从download中接受数据
	go download(ch)
	select {
	case <-ch:
		fmt.Println("从ch中读取数据，执行正常业务处理") // 如果从ch中读取到数据那么正常处理业务
	case <-timeout:
		fmt.Println("3秒内没有从ch中读取数据，执行超时处理")// 如果从timeout中读取到数据，那么download执行超时
	}
}
```
3、编程体：通过go协程输出100个以内的任意两个数之和，减少等待。

```go
func add(i, x, y int) {
	fmt.Printf("%d + %d = %d\n", x, y, x+y)
}
func a3() {
	for i := 1; i <= 100; i++ {
		x := rand.Intn(100)
		time.Sleep(time.Millisecond)
		y := rand.Intn(100)
		go add(i, x, y)
	}
}
```


## 二、go中的socket实现通信
> 知识速记

```go
// 共享数据机制-sync
func s1() {
	// sync.Mutex
	//mutex := sync.Mutex{}

}

// 上下文机制 - Context

// socket 原理
/*
	互联网TCP/IP四层模型
	四层：数据层（帧）、网络层（IP）、传输层（TCP/UDP）、应用层（HTTP）
	通信：封包和解包
	抽象：应用程序到应用程序、进程到进程、主机到主机、设备到设备

*/
```

1、代码示例clinet.go和server.go

> clinet.go

```go
package main
import (
	"bufio"
	"fmt"
	"net"
	"time"
)
func main() {
	// 与服务端建立连接
	conn, err := net.DialTimeout("tcp", "127.0.0.1:8899", time.Second)
	if err != nil {
		fmt.Printf("dial failed, err:%v\n", err)
		return
	}
	defer conn.Close()
	// 要发送的数据
	msg := []string{"hello world!", "golang", "c++", "python"}
	//  通过bufio方式发送
	writer := bufio.NewWriter(conn)
	for i, v := range msg {
		n, err := writer.Write([]byte(v))
		writer.Flush()
		if err != nil {
			fmt.Printf("write failed, err:%v\n", err)
			return
		}
		fmt.Printf("%d: write %d,bytes, data:%s\n", i, n, v)
		time.Sleep(time.Second)
	}
}
```
> server.go

```go
// go 基于 socket 的 tcp 编程
func HandleConn(conn net.Conn) {
	fmt.Println("accepted a new connection!")
	defer conn.Close()
	for {
		// 负责缓存接受的数据
		buf := make([]byte, 32)
		n, err := conn.Read(buf) //表面上是 阻塞读
		if err != nil {
			fmt.Println("read err:", err)
			break
		}
		if n == 0 {
			continue
		}
		// 打印客户端发送的数据
		fmt.Printf("recv client data: %s\n", string(buf[:n]))
		// 发送数据给客户端
		conn.Write([]byte("hello client"))
	}
}
func main() {
	// 开始监听 8899 端口
	listen, err := net.Listen("tcp", ":8899")
	if err != nil {
		fmt.Println("listen err:", err)
		return
	}
	// 循环接受连接
	for {
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("accept err:", err)
			break
		}
		// 处理连接
		go HandleConn(conn)
	}
}
```

2、go基于 socket 的 tcp 编程

1、连接建立问题
- 连接拒绝：网络ping不通、ip或port指定错误、server未启动
- listen backlog：增大server端listen backlog队列
- 网络延迟较大

2、读数据问题
- 无数据可读：goroutine阻塞即可
- 数据不足
- 超时

3、写数据问题
- 写阻塞
- 写入部分数据

4、线程安全问题



