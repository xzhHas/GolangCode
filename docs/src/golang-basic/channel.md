---
title: channel的底层实现原理？并发编程的利器？
shortTitle: 8.'通道'数据交换的核心结构
description: channe的数据结构，并发编程，通道
author: GolangCode
category:
  - Go
tags:
	- Go
date: 2024-04-05
star: true
order: -7
---

## 1.channe的数据结构

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	timer    *timer // timer feeding this chan
	elemtype *_type // element type
	sendx    uint   // 开始写的位置
	recvx    uint   // 开始读的位置
	recvq    waitq  // 添加读的等待队列
	sendq    waitq  // 添加写的等待队列
	lock mutex		// runtime.Mutex 保证线程安全
}

type waitq struct {
	first *sudog
	last  *sudog
}
```

对于channel我们可以缓存数据到里面，是因为有一个buf的数组用来缓冲数据，又因为channel可以同时提供读写功能，所以我们有sendx和recvx分别指向下一次写 和 下一次读的位置，buf、sendx、recvx就构成了一个环形数组。

因为channel是用在多个goroutine之间的通信，所以需要一把Mutex来保护读写关闭操作的并发安全，又因为channel提供一个读和写等待队列来帮助goroutine在未完成读写操作后，可以被阻塞挂起（将goroutine放入队列），然后等待channel通信来临再次被唤醒。

```go
type sudog struct {
	g *g	// 锁定的goroutine
	next *sudog
	prev *sudog
	elem unsafe.Pointer // data element (may point to stack)
	acquiretime int64
	releasetime int64
	ticket      uint32
	isSelect bool
	success bool
	waiters uint16
	parent   *sudog // semaRoot binary tree
	waitlink *sudog // g.waiting list or semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}

```

sudg是对阻塞挂起goroutine的一个封装，用多个sudg来构成等待队列。

## 2.channel的操作

### 2.1.channel写入

- channel中有读等待goroutine
  - 从recvq取出队列头部的sudg，将要写入的数据拷贝到sudg对应的elem容器上，唤醒sudg绑定的g
- channel中没有读等待goroutine，并且环形缓冲区数组里面有剩余空间
  - 将数据写到sendx的位置
- channel中没有读等待goroutine，并且无剩余空间存放数据
  - 获取一个sudg结构，绑定对应的goroutine，channel，ep指针
  - 将sudg放入channel的写等待队列
  - 使用runtime.gopark()挂起当前goroutine
- 写入的channel为nil
  - 对nil的channel进行写操作，会导致当前goroutine永久性挂起
  - 若在main 函数则直接fatal error
- channel已经关闭，还想进行写操作
  - 直接panic

### 2.2.channel读取

- channel中有写等待goroutine
  - 从写等待队列里面拿到头部sudog，进入recv流程
  - 如果channel无缓冲区，直接取出sudog里面的数据，并唤醒sudog里的g
  - 如果channel有缓冲区，先尝试读取环形数组recvx对应的元素，并将sudog中的元素写入到缓冲区，唤醒sudog对应的g
- channel中没有写等待goroutine，并且环形缓冲数组中有剩余元素
  - 读取recvx的数据，recvx++，qcount--
- channel中没有写等待goroutine，并且环形缓冲数组中无剩余元素
  - 获取一个sudog结构，绑定channel，goroutine，ep指针，将sudog放入channel的读等待队列，挂起当前goroutine
- 读取的channel为nil
  - main函数中直接fatal error；goroutine中永久挂起
- channel已经关闭，并且buf里面没有元素
  - 读取对应类型的零值

## 3.select

核心原理：按照随机顺序执行case，直到某个case操作完成，如果所有case的都没有完成，则看有没有default分支，如果有，就直接走default，防止阻塞。如果没有的话，就将当前goroutine加入到所有case对应channel的等待队列中，等待唤醒。如果当前goroutine被某一个case上的channel操作唤醒后，还需要将当前goroutine从所有case对应channel的等待独队列中剔除。

## 4.面试题

### 4.1.channel是线程安全的吗？为什么？

是线程安全的。

因为channel的底层数据结构都是用同一把runtime.Mutex来进行保护的，对channel的操作只有读、写、关闭三种操作。在Go语言中，channel主要是让goroutine之间传递数据和进行同步操作，通过channel，可以保证数据一致性和并发安全性。hchan的底层实现中，hchan结构体采用Mutex锁来保护数据读写安全。在对循环数组buf中的数据进行入队和出队操作时，必须先获取互斥锁，才能操作channel数据。

### 4.2.channel的底层实现原理

channel的底层数据结构叫做runtime.hchan，在hchan拥有一把runtime.Mutex来保证读写关闭操作逻辑的并发安全，通过读写指针和一个buf数组实现了一个环形缓冲队列，让channel拥有存储数据的能力；还拥有读写等待队列，当一个goroutine对channel进行读或写操作，操作无法及时完成的时候，可以进入队列等待，当前goroutine也被runtime.gopark挂起；读写操作也能取出等待队列里面的goroutine，通过runtime.goready将等待中的goroutine唤醒，等待GMP的调度。

### 4.3.对channel进行读写关闭操作？

1.对nil的channel进行读和写都会造成永久性阻塞，如果是在main函数 直接fatal error，关闭发生panic。
2.对不为nil，且未关闭的channel操作，读和写都有两种情况：

- 读操作	
  - 成功读取：如果channel中有数据，直接读取channel，如果此时等待队列recvxq里面有goroutine，那么需要将队列头部goroutine写入channel，唤醒这个goroutine；如果channel没有数据，就尝试从写等待队列中读取数据，并做对应的唤醒操作。
  - 阻塞挂起（读操作无法及时完成）：channel里面没有数据并且写等待队列为空，则当前goroutine加入等待队列中，并挂起，等待唤醒。
- 写操作
  - 成功写入：如果channel读等待队列不为空，则取头部goroutine，将数据直接复制给这个头部的goroutine，并将其唤醒，流程结束；否则就尝试将数据写入到channel环形缓冲中。
  - 阻塞挂起（写操作无法及时完成）：通道里面buf满了并且等待队列为空，则当前goroutine加入写等待队列中，并挂起，等待唤醒。

3.对已经关闭的channel进行写和关闭操作都会panic，而读取是直到读完channel中剩余的数据，还想读的话，就会获得零值。

## 5.算法题

### 5.1.select中case的使用

```go
func case1() {
	c1 := make(chan int)
	c2 := make(chan int)
	close(c1)
	close(c2)

	select {
	case <-c1:
		fmt.Println("c1")
	case c2 <- 1:
		fmt.Println("c2")
	default:
		fmt.Println("default")
	}
}
```

请问以上程序的输出结果？

select中的case调用是随机的，不确定到底先调用哪一个case使用。

```go
func case2() {
	c := make(chan int, 1)
	done := false
	for !done {
		select {
		case <-c:
			print(1)
			c = nil
		case c <- 1:
			print(2)
		default:
			print(3)
			done = true
		}
	}
}
```

有缓冲的channel，在写操作的时候，如果缓冲区未满那么直接写入数据，否则阻塞；在读操作的时候，如果有数据那么直接读取，否则阻塞。

### 5.2.有4个goroutine，按编号1、2、3、4循环打印

```go
func main() {
	ch := make([]chan int, 4)
	for i, _ := range ch {
		ch[i] = make(chan int)

		go func(i int) {
			for {
				v := <-ch[i]
				fmt.Println(v + 1)
				time.Sleep(time.Second)
				ch[(i+1)%4] <- (v + 1) % 4
			}
		}(i)
	}
	ch[0] <- 0
	select {}
}
```

## 6.最后

如果以上内容对你有帮助的话也可以关注我的公众号“GolangCode”，获取更多技术干货。

![GolangCode](https://cdn.golangcode.cn/images/202501171944968.png)
