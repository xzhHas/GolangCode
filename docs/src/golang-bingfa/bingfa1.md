---
title: 什么是并发编程？什么是并行，并发，串行？
shortTitle: 1.Go并发编程
description: 什么是并发编程？什么是并行，并发，串行？Go语言如何实现并发编程，以及实现的原理，goroutine的使用。
author: GolangCode
category:
  - Go
tag:
 - Go
date: 2024-05-07
star: true
---




> 前言
>
> 什么是并发编程？什么是并行，并发，串行？
>
> Go语言如何实现并发编程，以及实现的原理，goroutine的使用。
>
> runtime包、sync包的介绍。
>
> channel通道的使用，以及缓冲通道，定向通道。
>
> select语句，time包中和并发编程相关的函数介绍。
>
> CSP模型。

**推荐个人主页：** [席万里的个人空间](https://blog.csdn.net/m0_73337964?spm=1000.2115.3001.5343)

## 1、认识并发编程

### 1.1、并发性

#### 1.1.1、什么是并发

**并发性Concurrency是同时处理许多事情的能力。**

简单理解一下串行、并行、并发的运行情况。

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071511992.jpeg" alt="在这里插入图片描述" style="zoom:67%;" />


#### 1.1.2、进程、线程、协程

> 进程

1. **进程（Process）**：进程是操作系统中正在运行的一个程序的实例。每个进程都有自己独立的内存空间，包括代码、数据和堆栈等。进程之间相互独立，通信需要额外的机制。在Go中，可以通过`os/exec`包来创建和管理进程。

> 线程

1. **线程（Thread）**：线程是操作系统调度的最小执行单元。一个进程可以包含多个线程，它们共享进程的资源，如内存空间。不同线程之间可以通过共享内存进行通信。在Go语言中，可以通过`goroutine`来创建并发执行的线程。Goroutine是由Go语言运行时管理的轻量级线程，可以方便地创建、调度和管理。

> 协程

1. **协程（Coroutine）**：协程是一种用户态的轻量级线程，由用户程序来管理。和线程一样，协程也是并发执行的单元，但是它们可以在不同的执行路径中进行切换而不需要操作系统的参与，因此协程的切换开销比线程小。在Go语言中，goroutine可以看作是一种协程的实现，通过`go`关键字启动一个函数，该函数会以goroutine的形式并发执行。

### 1.2、Goroutine（协程）

#### 1.2.1、主协程

> 在Go语言中，主协程通常指的是程序的主线程，也就是程序的入口点。在Go程序中，主协程由`main()`函数所在的goroutine来表示。当程序开始执行时，会首先执行`main()`函数，这个函数所在的goroutine就是主协程。主协程负责初始化程序的状态、启动其他的goroutine，并在必要时等待其他goroutine的完成。
>
> 主协程的生命周期与整个程序的生命周期相同，当主协程执行完毕时，整个程序也就结束了。在主协程中，可以启动其他的goroutine来实现并发执行，也可以等待其他goroutine的完成或者通过通信与其他goroutine进行交互。
>

#### 1.2.2、启动多个goroutine

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// 启动多个goroutine
	for i := 0; i < 3; i++ {
		go worker(i)
	}
	// 等待一段时间，以便观察goroutine的执行
	time.Sleep(time.Second * 2)
}

func worker(id int) {
	fmt.Printf("Worker %d started\n", id)
	time.Sleep(time.Second)
	fmt.Printf("Worker %d completed\n", id)
}
```


### 1.3、并发模型

#### 1.3.1、线程模型

线程的实现模型主要有3个，分别是:用户级线程模型、内核级线程模型和两级线程模型。它们之间最大的差异就在于线程与内核调度实体( Kernel Scheduling Entity,简称KSE)之间的对应关系上。顾名思义，内核调度实体就是可以被内核的调度器调度的对象。在很多文献和书中，它也称为内核级线程，是操作系统内核的最小调度单元。

#### 1.3.2、GMP模型

在操作系统提供的内核线程之上，Go搭建了一个特有的两级线程模型。goroutine机制实现了M : N的线程模型，goroutine机制是协程（coroutine）的一种实现，golang内置的调度器，可以让多核CPU中每个CPU执行一个协程。

深入解析GMP模型：https://go.cyub.vip/gmp/gmp-model/

## 2、runtime包

### 2.1、runtime的主要函数

**runtime 调度器是个非常有用的东西，关于 runtime 包几个方法:**

- **NumCPU：**返回当前系统的 CPU 核数量
- **GOMAXPROCS：**设置最大的可同时使用的 CPU 核数

- **Gosched：**让当前线程让出 cpu 以让其它线程运行,它不会挂起当前线程，因此当前线程未来会继续执行

  > 这个函数的作用是让当前 goroutine 让出 CPU，当一个 goroutine 发生阻塞，Go 会自动地把与该 goroutine 处于同一系统线程的其他 goroutine 转移到另一个系统线程上去，以使这些 goroutine 不阻塞。

- **Goexit：**退出当前 goroutine(但是defer语句会照常执行)

- **NumGoroutine：**返回正在执行和排队的任务总数

  > runtime.NumGoroutine函数在被调用后，会返回系统中的处于特定状态的Goroutine的数量。这里的特指是指Grunnable\Gruning\Gsyscall\Gwaition。处于这些状态的Groutine即被看做是活跃的或者说正在被调度。
  > 注意：垃圾回收所在Groutine的状态也处于这个范围内的话，也会被纳入该计数器。

- **GOOS：**目标操作系统

- **runtime.GC：**会让运行时系统进行一次强制性的垃圾收集

  > 1.强制的垃圾回收：不管怎样，都要进行的垃圾回收。2.非强制的垃圾回收：只会在一定条件下进行的垃圾回收（即运行时，系统自上次垃圾回收之后新申请的堆内存的单元（也成为单元增量）达到指定的数值）。

- **GOROOT：**获取goroot目录

- **GOOS :** 查看目标操作系统 

### 2.2、功能代码示例

**1.获取goroot和os：**

```go
   //获取goroot目录：
    fmt.Println("GOROOT-->",runtime.GOROOT())
   
    //获取操作系统
    fmt.Println("os/platform-->",runtime.GOOS) // GOOS--> darwin，mac系统
```

**2.获取CPU数量，和设置CPU数量：**

```go
func init(){
    //1.获取逻辑cpu的数量
    fmt.Println("CPU的核数：",runtime.NumCPU())
    //2.设置go程序执行的最大的：[1,256]
    n := runtime.GOMAXPROCS(runtime.NumCPU())
    fmt.Println(n)
}
```



**3.Gosched()：**

```go
func main() {
    go func() {
        for i := 0; i < 5; i++ {
            fmt.Println("goroutine。。。")
        }

    }()

    for i := 0; i < 4; i++ {
        //让出时间片，先让别的协议执行，它执行完，再回来执行此协程
        runtime.Gosched()
        fmt.Println("main。。")
    }
}

```

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071511192.jpeg" alt="在这里插入图片描述" style="zoom:50%;" />


**4.Goexit的使用（终止协程）**

```go
func main() {
    //创建新建的协程
    go func() {
        fmt.Println("goroutine开始。。。")

        //调用了别的函数
        fun()

        fmt.Println("goroutine结束。。")
    }() //别忘了()

    //睡一会儿，不让主协程结束
    time.Sleep(3*time.Second)
}

func fun() {
    defer fmt.Println("defer。。。")

    //return           //终止此函数
    runtime.Goexit() //终止所在的协程

    fmt.Println("fun函数。。。")
}
```

## 3、sync包

### 3.1、临界资源安全问题

> 并发本身并不复杂，但是因为有了资源竞争的问题，就使得我们开发出好的并发程序变得复杂起来，因为会引起很多莫名其妙的问题。
>
> **如果多个goroutine在访问同一个数据资源的时候，其中一个线程修改了数据，那么这个数值就被修改了，对于其他的goroutine来讲，这个数值可能是不对的。**

#### 3.1.1、代码示例

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

var ticket = 10 // 10张票
func main() {
	/*
	   4个goroutine，模拟4个售票口，4个子程序操作同一个共享数据。
	*/
	go saleTickets("售票口1") // g1,100
	go saleTickets("售票口2") // g2,100
	go saleTickets("售票口3") //g3,100
	go saleTickets("售票口4") //g4,100
	time.Sleep(5 * time.Second)
}
func saleTickets(name string) {
	rand.Seed(time.Now().UnixNano())
	for { //ticket=1
		if ticket > 0 { //g1,g3,g2,g4
			//睡眠
			time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
			// g1 ,g3, g2,g4
			fmt.Println(name, "售出：", ticket)
			ticket--
		} else {
			fmt.Println(name, "售罄，没有票了。。")
			break
		}
	}
}
```

> 运行结果
>
> ![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181900109.jpeg)


**这就是临界资源的不安全问题。某一个goroutine在访问某个数据资源的时候，按照数值，已经判断好了条件，然后又被其他的goroutine抢占了资源，并修改了数值，等这个goroutine再继续访问这个数据的时候，数值已经不对了。**

#### 3.1.2、解决方案

> 要想解决临界资源安全的问题，很多编程语言的解决方案都是同步。通过上锁的方式，某一时间段，只能允许一个goroutine来访问这个共享数据，当前goroutine访问完毕，解锁后，其他的goroutine才能来访问。
>
> **我们可以借助于sync包下的锁操作。**

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

var ticket = 10
var wg sync.WaitGroup
var matex sync.Mutex // 创建锁头
func main() {
	wg.Add(4)
	go saleTickets("售票口1") // g1,100
	go saleTickets("售票口2") // g2,100
	go saleTickets("售票口3") //g3,100
	go saleTickets("售票口4") //g4,100
	wg.Wait()              // main要等待。。。
}

func saleTickets(name string) {
	rand.Seed(time.Now().UnixNano())
	defer wg.Done()
	for { //ticket=1
		matex.Lock()
		if ticket > 0 { //g1,g3,g2,g4
			//睡眠
			time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
			// g1 ,g3, g2,g4
			fmt.Println(name, "售出：", ticket) // 1 , 0, -1 , -2
			ticket--                         //0 , -1 ,-2 , -3
		} else {
			matex.Unlock() //解锁
			fmt.Println(name, "售罄，没有票了。。")
			break
		}
		matex.Unlock() //解锁
	}
}
```

> 运行结果
>
> <img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071511933.jpeg" alt="在这里插入图片描述" style="zoom: 50%;" />


可见，成功解决这个问题了。

>  **注意：不要以共享内存的方式去通信，而要以通信的方式去共享内存。**
>
> 在Go语言中并不鼓励用锁保护共享状态的方式在不同的Goroutine中分享信息(以共享内存的方式去通信)。而是鼓励通过**channel**将共享状态或共享状态的变化在各个Goroutine之间传递（以通信的方式去共享内存），这样同样能像用锁一样保证在同一的时间只有一个Goroutine访问共享状态。
>
> 当然，在主流的编程语言中为了保证多线程之间共享数据安全性和一致性，都会提供一套基本的同步工具集，如锁，条件变量，原子操作等等。Go语言标准库也毫不意外的提供了这些同步机制，使用方式也和其他语言也差不多。

### 3.2、sync包

#### 3.2.1、WaitGroup

1. WaitGroup

   WaitGroup，同步等待组。

   在类型上，它是一个结构体。一个WaitGroup的用途是等待一个goroutine的集合执行完成。主goroutine调用了Add()方法来设置要等待的goroutine的数量。然后，每个goroutine都会执行并且执行完成后调用Done()这个方法。与此同时，可以使用Wait()方法来阻塞，直到所有的goroutine都执行完成。

2. Add()方法

   `func (wg *WaitGroup) Add(delta int)`

   > Add这个方法，用来设置到WaitGroup的计数器的值。我们可以理解为每个waitgroup中都有一个计数器 用来表示这个同步等待组中要执行的goroutin的数量。
   >
   > 如果计数器的数值变为0，那么就表示等待时被阻塞的goroutine都被释放，如果计数器的数值为负数，那么就会引发恐慌，程序就报错了。

3. Done()方法

   `func (wg *WaitGroup) Done()`

   > Done()方法，就是当WaitGroup同步等待组中的某个goroutine执行完毕后，设置这个WaitGroup的counter数值减1。
   >
   > ```go
   > func (wg *WaitGroup) Done() {
   > 	wg.Add(-1)
   > }
   > ```

4. Wait()方法

   `func (wg *WaitGroup) Wait()`

   > Wait()方法，表示让当前的goroutine等待，进入阻塞状态。一直到WaitGroup的计数器为零。才能解除阻塞， 这个goroutine才能继续执行。

5. 代码示例

   >  我们创建并启动两个goroutine，来打印数字和字母，并在main goroutine中，将这两个子goroutine加入到一个WaitGroup中，同时让main goroutine进入Wait()，让两个子goroutine先执行。当每个子goroutine执行完毕后，调用Done()方法，设置WaitGroup的counter减1。当两条子goroutine都执行完毕后，WaitGroup中的counter的数值为零，解除main goroutine的阻塞。

   ```go
   package main
   
   import (
   	"fmt"
   	"sync"
   )
   
   var wg sync.WaitGroup // 创建同步等待组对象
   func main() {
   	/*
   	   WaitGroup：同步等待组
   	       可以使用Add(),设置等待组中要 执行的子goroutine的数量，
   	       在main 函数中，使用wait(),让主程序处于等待状态。直到等待组中子程序执行完毕。解除阻塞
   	       子gorotuine对应的函数中。wg.Done()，用于让等待组中的子程序的数量减1
   	*/
   	//设置等待组中，要执行的goroutine的数量
   	wg.Add(2)
   	go fun1()
   	go fun2()
   	fmt.Println("main进入阻塞状态。。。等待wg中的子goroutine结束。。")
   	wg.Wait() //表示main goroutine进入等待，意味着阻塞
   	fmt.Println("main，解除阻塞。。")
   
   }
   func fun1() {
   	for i := 1; i <= 6; i++ {
   		fmt.Println("fun1.。。i:", i)
   	}
   	wg.Done() //给wg等待中的执行的goroutine数量减1.同Add(-1)
   }
   func fun2() {
   	defer wg.Done()
   	for j := 1; j <= 6; j++ {
   		fmt.Println("\tfun2..j,", j)
   	}
   }
   ```

   > 运行结果
   >
   > ![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181900303.jpeg)


####  3.2.2、互斥锁

1. Mutex（互斥锁）

   > 通过上一小节，我们知道了在并发程序中，会存在临界资源问题。就是当多个协程来访问共享的数据资源，那么这个共享资源是不安全的。为了解决协程同步的问题我们使用了channel，但是Go语言也提供了传统的同步工具。
   >
   > 什么是锁呢？就是某个协程（线程）在访问某个资源时先锁住，防止其它协程的访问，等访问完毕解锁后其他协程再来加锁进行访问。一般用于处理并发中的临界资源问题。
   >
   > Go语言包中的 sync 包提供了两种锁类型：sync.Mutex 和 sync.RWMutex。
   >
   > Mutex 是最简单的一种锁类型，互斥锁，同时也比较暴力，当一个 goroutine 获得了 Mutex 后，其他 goroutine 就只能乖乖等到这个 goroutine 释放该 Mutex。
   >
   > 每个资源都对应于一个可称为 “互斥锁” 的标记，这个标记用来保证在任意时刻，只能有一个协程（线程）访问该资源。其它的协程只能等待。
   >
   > 互斥锁是传统并发编程对共享资源进行访问控制的主要手段，它由标准库sync中的Mutex结构体类型表示。sync.Mutex类型只有两个公开的指针方法，Lock和Unlock。Lock锁定当前的共享资源，Unlock进行解锁。
   >
   > 在使用互斥锁时，一定要注意：对资源操作完成后，一定要解锁，否则会出现流程执行异常，死锁等问题。通常借助defer。锁定后，立即使用defer语句保证互斥锁及时解锁。

2. Lock()

   `func (m *Mutex) Lock()`

   > Lock()这个方法，锁定m。如果该锁已在使用中，则调用goroutine将阻塞，直到互斥体可用。

3. Unlock()

   `func (m *Mutex) Unlock()`

   > Unlock()方法，解锁m。如果m未在要解锁的条目上锁定，则为运行时错误。
   >
   > 锁定的互斥体不与特定的goroutine关联。允许一个goroutine锁定互斥体，然后安排另一个goroutine解锁互斥体。

4. 示例代码

   > 我们针对于上次课程汇总，使用goroutine，模拟4个售票口出售火车票的案例。4个售票口同时卖票，会发生临界资源数据安全问题。我们使用互斥锁解决一下。(Go语言推崇的是使用Channel来实现数据共享，但是也还是提供了传统的同步处理方式)

   ```go
   package main
   
   import (
   	"fmt"
   	"math/rand"
   	"sync"
   	"time"
   )
   var ticket = 10 //100张票
   var mutex sync.Mutex //创建锁头
   var wg sync.WaitGroup //同步等待组对象
   func main() {
   	/*
   	   4个goroutine，模拟4个售票口，
   	   在使用互斥锁的时候，对资源操作完，一定要解锁。否则会出现程序异常，死锁等问题。
   	   defer语句
   	*/
   	wg.Add(4)
   	go saleTickets("售票口1")
   	go saleTickets("售票口2")
   	go saleTickets("售票口3")
   	go saleTickets("售票口4")
   
   	wg.Wait() //main要等待
   	fmt.Println("程序结束了。。。")
   
   	//time.Sleep(5*time.Second)
   }
   func saleTickets(name string) {
   	rand.Seed(time.Now().UnixNano())
   	defer wg.Done()
   	for {
   		//上锁
   		mutex.Lock()    //g2
   		if ticket > 0 { //ticket 1 g1
   			time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
   			fmt.Println(name, "售出：", ticket) // 1
   			ticket--                         // 0
   		} else {
   			mutex.Unlock() //条件不满足，也要解锁
   			fmt.Println(name, "售罄，没有票了。。")
   			break
   		}
   		mutex.Unlock() //解锁
   	}
   }
   ```

   > 运行结果
   >
   > ![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181900422.jpeg)


#### 3.2.3、读写锁

1. RWMutx

   > 通过对互斥锁的学习，我们已经知道了锁的概念以及用途。主要是用于处理并发中的临界资源问题。
   >
   > Go语言包中的 sync 包提供了两种锁类型：sync.Mutex 和 sync.RWMutex。其中RWMutex是基于Mutex实现的，只读锁的实现使用类似引用计数器的功能。
   >
   > RWMutex是读/写互斥锁。锁可以由任意数量的读取器或单个编写器持有。RWMutex的零值是未锁定的mutex。
   >
   > 如果一个goroutine持有一个RWMutex进行读取，而另一个goroutine可能调用lock，那么在释放初始读取锁之前，任何goroutine都不应该期望能够获取读取锁。特别是，这禁止递归读取锁定。这是为了确保锁最终可用；被阻止的锁调用会将新的读卡器排除在获取锁之外。

   我们怎么理解读写锁呢？

   1. 同时只能有一个 goroutine 能够获得写锁定。
   2. 同时可以有任意多个 gorouinte 获得读锁定。
   3. 同时只能存在写锁定或读锁定（读和写互斥）。

   所以，RWMutex这个读写锁，该锁可以加多个读锁或者一个写锁，其经常用于读次数远远多于写次数的场景。

   读写锁的写锁只能锁定一次，解锁前不能多次锁定，读锁可以多次，但读解锁次数最多只能比读锁次数多一次，一般情况下我们不建议读解锁次数多余读锁次数。

   基本遵循两大原则：

   1、可以随便读，多个goroutine同时读。

   2、写的时候，啥也不能干。不能读也不能写。

   读写锁即是针对于读写操作的互斥锁。它与普通的互斥锁最大的不同就是，它可以分别针对读操作和写操作进行锁定和解锁操作。读写锁遵循的访问控制规则与互斥锁有所不同。在读写锁管辖的范围内，它允许任意个读操作的同时进行。但是在同一时刻，它只允许有一个写操作在进行。

   并且在某一个写操作被进行的过程中，读操作的进行也是不被允许的。也就是说读写锁控制下的多个写操作之间都是互斥的，并且写操作与读操作之间也都是互斥的。但是，多个读操作之间却不存在互斥关系。

2. RLock()

   ```go
   func (rw *RWMutex) RLock()
   ```

   > 读锁，当有写锁时，无法加载读锁，当只有读锁或者没有锁时，可以加载读锁，读锁可以加载多个，所以适用于“读多写少”的场景。

3. RUnlock()

   ```go
   func (rw *RWMutex) RUnlock()
   ```

   > 读锁解锁，RUnlock 撤销单次RLock调用，它对于其它同时存在的读取器则没有效果。若rw并没有为读取而锁定，调用RUnlock就会引发一个运行时错误。

4. Lock()

   ```go
   func (rw *RWMutex) Lock()
   ```

   > 写锁，如果在添加写锁之前已经有其他的读锁和写锁，则Lock就会阻塞直到该锁可用，为确保该锁最终可用，已阻塞的Lock调用会从获得的锁中排除新的读取锁，即写锁权限高于读锁，有写锁时优先进行写锁定。

5. Unlock()

   ```go
   func (rw *RWMutex) Unlock()
   ```

   > 写锁解锁，如果没有进行写锁定，则就会引起一个运行时错误。

6. 示例代码

   ```go
   package main
   
   import (
   	"fmt"
   	"sync"
   	"time"
   )
   var rwMutex *sync.RWMutex
   var wg *sync.WaitGroup
   func main() {
   	rwMutex = new(sync.RWMutex)
   	wg = new(sync.WaitGroup)
   	wg.Add(3)
   	go writeData(1)
   	go readData(2)
   	go writeData(3)
   	wg.Wait()
   	fmt.Println("main..over...")
   }
   
   func writeData(i int) {
   	defer wg.Done()
   	fmt.Println(i, "开始写：write start。。")
   	rwMutex.Lock() //写操作上锁
   	fmt.Println(i, "正在写：writing。。。。")
   	time.Sleep(3 * time.Second)
   	rwMutex.Unlock()
   	fmt.Println(i, "写结束：write over。。")
   }
   
   func readData(i int) {
   	defer wg.Done()
   	fmt.Println(i, "开始读：read start。。")
   	rwMutex.RLock() //读操作上锁
   	fmt.Println(i, "正在读取数据：reading。。。")
   	time.Sleep(3 * time.Second)
   	rwMutex.RUnlock() //读操作解锁
   	fmt.Println(i, "读结束：read over。。。")
   }
   ```

   > 运行结果
   >
   > ![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181900641.jpeg)

   >
   > 注意：
   >
   > 1. 读锁不能阻塞读锁
   > 2. 读锁需要阻塞写锁，直到所有读锁都释放
   > 3. 写锁需要阻塞读锁，直到所有写锁都释放
   > 4. 写锁需要阻塞写锁

## 4、channel通道

### 4.1、channel

通道可以被认为是Goroutines通信的管道。类似于管道中的水从一端到另一端的流动，数据可以从一端发送到另一端，通过通道接收。

在前面讲Go语言的并发时候，我们就说过，当多个Goroutine想实现共享数据的时候，虽然也提供了传统的同步机制，但是Go语言强烈建议的是使用Channel通道来实现Goroutines之间的通信。

```go
“不要通过共享内存来通信，而应该通过通信来共享内存” 
```

Go语言中，**要传递某个数据给另一个goroutine(协程)，可以把这个数据封装成一个对象，然后把这个对象的指针传入某个channel中，另外一个goroutine从这个channel中读出这个指针，并处理其指向的内存对象**。Go从语言层面保证**同一个时间只有一个goroutine能够访问channel里面的数据**，为开发者提供了一种优雅简单的工具，所以Go的做法就是使用channel来通信，通过通信来传递内存数据，使得内存数据在不同的goroutine中传递，而不是使用共享内存来通信。

#### 4.1.1、声明通道

```go
//声明通道
var 通道名 chan 数据类型
//创建通道：如果通道为nil(就是不存在)，就需要先创建通道
通道名 = make(chan 数据类型)
//简短方法
通道名 := make(chan int) 
```

#### 4.1.2、channel 的数据类型

> channel是引用类型的数据，在作为参数传递的时候，传递的是内存地址。

在Go语言中，channel可以用于传递任何类型的数据，包括内置的基本数据类型、复合数据类型和自定义数据类型。

1. **基本数据类型**：
   - 整数类型：`int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`
   - 浮点数类型：`float32`, `float64`
   - 布尔类型：`bool`
   - 字符串类型：`string`

```go
ch := make(chan int)
ch <- 10
```

2. **复合数据类型**：
   - 数组：`[...]int`, `[5]int`
   - 切片：`[]int`, `[]string`
   - 映射：`map[string]int`, `map[int]string`
   - 结构体：自定义的结构体类型

```go
ch := make(chan []int)
ch <- []int{1, 2, 3, 4, 5}
```

3. **接口类型**：
   - 空接口：`any`
   - 具体接口类型：定义了一组方法的接口类型

```go
ch := make(chan any)
ch <- "hello"
ch <- 42
```

4. **自定义数据类型**：
   - 通过`type`关键字定义的任何自定义数据类型

```go
type Person struct {
    Name string
    Age  int
}

ch := make(chan Person)
ch <- Person{Name: "Alice", Age: 30}
```

> Channel通道在使用的时候，有以下几个注意点：
>
> - 1.用于goroutine，传递消息的。
> - 2.通道，每个都有相关联的数据类型, nil chan，不能使用，类似于nil map，不能直接存储键值对
> - 3.使用通道传递数据：<- chan <- data,发送数据到通道。向通道中写数据 data <- chan,从通道中获取数据。从通道中读数据
> - 4.阻塞： 发送数据：chan <- data,阻塞的，直到另一条goroutine，读取数据来解除阻塞 读取数据：data <- chan,也是阻塞的。直到另一条goroutine，写出数据解除阻塞。
> - 5.本身channel就是同步的，意味着同一时间，只能有一条goroutine来操作。
>
> **通道是goroutine之间的连接，所以通道的发送和接收必须处在不同的goroutine中。**

#### 4.1.3、channel的使用方法

> 关于channel的读取和使用。

```go
package main

import (
	"fmt"
)

func main() {
	// 创建一个可以传递整数类型数据的channel
	ch := make(chan int)
	// 启动一个goroutine进行写入数据
	go func() {
		for i := 1; i <= 3; i++ {
			fmt.Printf("Writing %d to channel\n", i)
			ch <- i // 写入数据到channel
		}
		close(ch) // 关闭channel，表示写入完成
	}()
	// 从channel中读取数据
	for {
		num, ok := <-ch // 从channel中读取数据
		if !ok {
			break // 如果channel已关闭，则退出循环
		}
		fmt.Printf("Reading %d from channel\n", num)
	}
	// 从channel中读取数据
	//for num := range ch {
	//	fmt.Printf("Reading %d from channel\n", num)
	//}
	fmt.Println("successful！")
}
```

> 运行结果
>
> ![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181900852.jpeg)


#### 4.1.4、死锁问题

> 使用通道时要考虑的一个重要因素是死锁。如果Goroutine在一个通道上发送数据，那么预计其他的Goroutine应该接收数据。如果这种情况不发生，那么程序将在运行时出现死锁。
>
> 类似地，如果Goroutine正在等待从通道接收数据，那么另一些Goroutine将会在该通道上写入数据，否则程序将会死锁。

```go
package main

import "fmt"

func main() {
	ch := make(chan int)
	ch <- 5
	close(ch)

	num, ok := <-ch
	if ok != true {
		fmt.Println(ok, num)
	} else {
		fmt.Println(ok, num)
	}
}
```

> 运行结果
>
> ![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181900590.jpeg)


对上说问题分析：

> 这段代码报错是因为在主goroutine中创建了一个无缓冲的channel `ch`，然后立即尝试向这个无缓冲的channel写入数据，但是没有任何其他goroutine在读取这个channel，因此会导致程序死锁。
>
> 在Go语言中，无缓冲的channel的发送和接收操作是同步的。当我们向无缓冲的channel发送数据时，如果没有其他goroutine在等待从该channel中接收数据，发送操作会阻塞当前goroutine，直到有其他goroutine准备好从该channel中接收数据；同样，当我们从无缓冲的channel接收数据时，如果没有其他goroutine在等待向该channel发送数据，接收操作也会阻塞当前goroutine，直到有其他goroutine准备好向该channel发送数据。

解决方案：

1、启动一个goroutine来接收数据：

```go
	ch := make(chan int)
	go func() {
		ch <- 5
		close(ch)
	}()
```

2、使用带缓冲的channel：

```go
	ch := make(chan int, 1) // 创建一个带缓冲的channel
	ch <- 5
	close(ch)
```

#### 4.1.5、time包中的有关通道的函数

主要就是定时器，标准库中的Timer让用户可以定义自己的超时逻辑，尤其是在应对select处理多个channel的超时、单channel读写的超时等情形时尤为方便。

Timer是一次性的时间触发事件，这点与Ticker不同，Ticker是按一定时间间隔持续触发时间事件。

Timer常见的创建方式：

```go
t:= time.NewTimer(d)
t:= time.AfterFunc(d, f)
c:= time.After(d)
```

虽然说创建方式不同，但是原理是相同的。

Timer有3个要素：

```go
定时时间：就是那个d
触发动作：就是那个f
时间channel： 也就是t.C
```

##### 4.1.5.1、time.NewTimer()

> NewTimer()创建一个新的计时器，该计时器将在其通道上至少持续d之后发送当前时间。

示例代码：

```go
package main

import (
    "time"
    "fmt"
)

func main() {
    /*
        1.func NewTimer(d Duration) *Timer
            创建一个计时器：d时间以后触发，go触发计时器的方法比较特别，就是在计时器的channel中发送值
     */
    //新建一个计时器：timer
    timer := time.NewTimer(3 * time.Second)
    fmt.Printf("%T\n", timer) //*time.Timer
    fmt.Println(time.Now())   //2019-08-15 10:41:21.800768 +0800 CST m=+0.000461190

    //此处在等待channel中的信号，执行此段代码时会阻塞3秒
    ch2 := timer.C     //<-chan time.Time
    fmt.Println(<-ch2) //2019-08-15 10:41:24.803471 +0800 CST m=+3.003225965
}
```

##### 4.1.5.2、timer.Stop

计时器停止。

示例代码：

```go
package main

import (
    "time"
    "fmt"
)

func main() {
    /*
        1.func NewTimer(d Duration) *Timer
            创建一个计时器：d时间以后触发，go触发计时器的方法比较特别，就是在计时器的channel中发送值
     */
    //新建一个计时器：timer
    //timer := time.NewTimer(3 * time.Second)
    //fmt.Printf("%T\n", timer) //*time.Timer
    //fmt.Println(time.Now())   //2019-08-15 10:41:21.800768 +0800 CST m=+0.000461190
    //
    ////此处在等待channel中的信号，执行此段代码时会阻塞3秒
    //ch2 := timer.C     //<-chan time.Time
    //fmt.Println(<-ch2) //2019-08-15 10:41:24.803471 +0800 CST m=+3.003225965
    fmt.Println("-------------------------------")
    //新建计时器，一秒后触发
    timer2 := time.NewTimer(5 * time.Second)
    //新开启一个线程来处理触发后的事件
    go func() {
        //等触发时的信号
        <-timer2.C
        fmt.Println("Timer 2 结束。。")
    }()
    //由于上面的等待信号是在新线程中，所以代码会继续往下执行，停掉计时器
    time.Sleep(3*time.Second)
    stop := timer2.Stop()
    if stop {
        fmt.Println("Timer 2 停止。。")
    }
}
```

##### 4.1.5.3、time.After()

> 在等待持续时间之后，然后在返回的通道上发送当前时间。它相当于NewTimer(d).C。在计时器触发之前，垃圾收集器不会恢复底层计时器。如果效率有问题，使用NewTimer代替，并调用Timer。如果不再需要计时器，请停止。

示例代码：

```go
package main

import (
    "time"
    "fmt"
)

func main() {
    /*
        func After(d Duration) <-chan Time
            返回一个通道：chan，存储的是d时间间隔后的当前时间。
     */
    ch1 := time.After(3 * time.Second) //3s后
    fmt.Printf("%T\n", ch1) // <-chan time.Time
    fmt.Println(time.Now()) //2019-08-15 09:56:41.529883 +0800 CST m=+0.000465158
    time2 := <-ch1
    fmt.Println(time2) //2019-08-15 09:56:44.532047 +0800 CST m=+3.002662179
}
```

### 4.2、缓冲通道

#### 4.2.1、非缓冲通道

> 之前学习的所有通道基本上都没有缓冲。发送和接收到一个未缓冲的通道是阻塞的。
>
> 一次发送操作对应一次接收操作，对于一个goroutine来讲，它的一次发送，在另一个goroutine接收之前都是阻塞的。同样的，对于接收来讲，在另一个goroutine发送之前，它也是阻塞的。

#### 4.2.2、缓冲通道

> 缓冲通道就是指一个通道，带有一个缓冲区。发送到一个缓冲通道只有在缓冲区满时才被阻塞。类似地，从缓冲通道接收的信息只有在缓冲区为空时才会被阻塞。
>
> **可以通过将额外的容量参数传递给make函数来创建缓冲通道，该函数指定缓冲区的大小。**

语法：

```go
ch := make(chan type, capacity)  
```

> 上述语法的容量应该大于0，以便通道具有缓冲区。默认情况下，无缓冲通道的容量为0，因此在之前创建通道时省略了容量参数。

#### 4.2.3、示例代码

以下的代码中，chan通道，是带有缓冲区的。

```go
package main

import (
    "fmt"
    "strconv"
    "time"
)

func main() {
    /*
    非缓存通道：make(chan T)
    缓存通道：make(chan T ,size)
        缓存通道，理解为是队列：
    非缓存，发送还是接受，都是阻塞的
    缓存通道,缓存区的数据满了，才会阻塞状态。。
     */
    ch1 := make(chan int)           //非缓存的通道
    fmt.Println(len(ch1), cap(ch1)) //0 0
    //ch1 <- 100//阻塞的，需要其他的goroutine解除阻塞，否则deadlock

    ch2 := make(chan int, 5)        //缓存的通道，缓存区大小是5
    fmt.Println(len(ch2), cap(ch2)) //0 5
    ch2 <- 100                      //
    fmt.Println(len(ch2), cap(ch2)) //1 5

    //ch2 <- 200
    //ch2 <- 300
    //ch2 <- 400
    //ch2 <- 500
    //ch2 <- 600
    fmt.Println("--------------")
    ch3 := make(chan string, 4)
    go sendData3(ch3)
    for {
        time.Sleep(1*time.Second)
        v, ok := <-ch3
        if !ok {
            fmt.Println("读完了，，", ok)
            break
        }
        fmt.Println("\t读取的数据是：", v)
    }
    fmt.Println("main...over...")
}
func sendData3(ch3 chan string) {
    for i := 0; i < 10; i++ {
        ch3 <- "数据" + strconv.Itoa(i)
        fmt.Println("子goroutine，写出第", i, "个数据")
    }
    close(ch3)
}
```

### 4.3、定向通道

#### 4.3.1、双向通道

> 通道，channel，是用于实现goroutine之间的通信的。一个goroutine可以向通道中发送数据，另一条goroutine可以从该通道中获取数据。截止到现在我们所学习的通道，都是既可以发送数据，也可以读取数据，我们又把这种通道叫做双向通道。

```go
data := <- a // read from channel a  
a <- data // write to channel a
```

#### 4.3.2、单向通道

> 单向通道，也就是定向通道。
>
> 之前我们学习的通道都是双向通道，我们可以通过这些通道接收或者发送数据。我们也可以创建单向通道，这些通道只能发送或者接收数据。

双向通道，实例代码：

```go
package main

import "fmt"

func main()  {
    /*
    双向：
        chan T -->
            chan <- data,写出数据，写
            data <- chan,获取数据，读
    单向：定向
        chan <- T,
            只支持写，
        <- chan T,
            只读
     */
    ch1 := make(chan string) // 双向，可读，可写
    done := make(chan bool)
    go sendData(ch1, done)
    data :=<- ch1 //阻塞
    fmt.Println("子goroutine传来：", data)
    ch1 <- "我是main。。" // 阻塞
    <-done
    fmt.Println("main...over....")
}
//子goroutine-->写数据到ch1通道中
//main goroutine-->从ch1通道中取
func sendData(ch1 chan string, done chan bool)  {
    ch1 <- "我是小明"// 阻塞
    data := <-ch1 // 阻塞
    fmt.Println("main goroutine传来：",data)

    done <- true
}
```

创建仅能发送数据的通道。

示例代码：

```go
package main

import "fmt"

func main()  {
    /*
        单向：定向
        chan <- T,
            只支持写，
        <- chan T,
            只读

        用于参数传递：
     */
    ch1 := make(chan int)//双向，读，写
    //ch2 := make(chan <- int) // 单向，只写，不能读
    //ch3 := make(<- chan int) //单向，只读，不能写
    //ch1 <- 100
    //data :=<-ch1
    //ch2 <- 1000
    //data := <- ch2
    //fmt.Println(data)
    //  <-ch2 //invalid operation: <-ch2 (receive from send-only type chan<- int)
    //ch3 <- 100
    //  <-ch3
    //  ch3 <- 100 //invalid operation: ch3 <- 100 (send to receive-only type <-chan int)

    //go fun1(ch2)
    go fun1(ch1)
    data:= <- ch1
    fmt.Println("fun1中写出的数据是：",data)

    //fun2(ch3)
    go fun2(ch1)
    ch1 <- 200
    fmt.Println("main。。over。。")
}
//该函数接收，只写的通道
func fun1(ch chan <- int){
    // 函数内部，对于ch只能写数据，不能读数据
    ch <- 100
    fmt.Println("fun1函数结束。。")
}
func fun2(ch <-chan int){
    //函数内部，对于ch只能读数据，不能写数据
    data := <- ch
    fmt.Println("fun2函数，从ch中读取的数据是：",data)
}
```

### 4.4、select语句

> select 是 Go 中的一个控制结构。select 语句类似于 switch 语句，但是select会随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。

select语句的语法结构和switch语句很相似，也有case语句和default语句：

```go
select {
    case communication clause  :
       statement(s);      
    case communication clause  :
       statement(s); 
    /* 你可以定义任意数量的 case */
    default : /* 可选 */
       statement(s);
}
```

说明：

- 每个case都必须是一个通信

- 所有channel表达式都会被求值

- 所有被发送的表达式都会被求值

- 如果有多个case都可以运行，select会随机公平地选出一个执行。其他不会执行。

- 否则：

  如果有default子句，则执行该语句。

  如果没有default字句，select将阻塞，直到某个通信可以运行；Go不会重新对channel或值进行求值。

示例代码：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    /*
    分支语句：if，switch，select
    select 语句类似于 switch 语句，
        但是select会随机执行一个可运行的case。
        如果没有case可运行，它将阻塞，直到有case可运行。
     */
    ch1 := make(chan int)
    ch2 := make(chan int)

    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- 200
    }()
    go func() {
        time.Sleep(2 * time.Second)
        ch1 <- 100
    }()

    select {
    case num1 := <-ch1:
        fmt.Println("ch1中取数据。。", num1)
    case num2, ok := <-ch2:
        if ok {
            fmt.Println("ch2中取数据。。", num2)
        }else{
            fmt.Println("ch2通道已经关闭。。")
        }

    }
}
```

select语句结合time包的和chan相关函数，示例代码：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    //go func() {
    //  ch1 <- 100
    //}()

    select {
    case <-ch1:
        fmt.Println("case1可以执行。。")
    case <-ch2:
        fmt.Println("case2可以执行。。")
    case <-time.After(3 * time.Second):
        fmt.Println("case3执行。。timeout。。")

    //default:
    //  fmt.Println("执行了default。。")
    }
}
```

### 4.5、CSP模型

> go语言的最大两个亮点，一个是goroutine，一个就是chan了。二者合体的典型应用CSP，基本就是大家认可的并行开发神器，简化了并行程序的开发难度，我们来看一下CSP。

#### 4.5.1、CSP是什么

CSP 是 Communicating Sequential Process 的简称，中文可以叫做通信顺序进程，是一种并发编程模型，是一个很强大的并发数据模型，是上个世纪七十年代提出的，用于描述两个独立的并发实体通过共享的通讯 channel(管道)进行通信的并发模型。相对于Actor模型，CSP中channel是第一类对象，它不关注发送消息的实体，而关注与发送消息时使用的channel。

严格来说，CSP 是一门形式语言（类似于 ℷ calculus），用于描述并发系统中的互动模式，也因此成为一众面向并发的编程语言的理论源头，并衍生出了 Occam/Limbo/Golang…

而具体到编程语言，如 Golang，其实只用到了 CSP 的很小一部分，即理论中的 Process/Channel（对应到语言中的 goroutine/channel）：这两个并发原语之间没有从属关系， Process 可以订阅任意个 Channel，Channel 也并不关心是哪个 Process 在利用它进行通信；Process 围绕 Channel 进行读写，形成一套有序阻塞和可预测的并发模型。

#### 4.5.2、Golang CSP

与主流语言通过共享内存来进行并发控制方式不同，Go 语言采用了 CSP 模式。这是一种用于描述两个独立的并发实体通过共享的通讯 Channel（管道）进行通信的并发模型。

Golang 就是借用CSP模型的一些概念为之实现并发进行理论支持，其实从实际上出发，go语言并没有，完全实现了CSP模型的所有理论，仅仅是借用了 process和channel这两个概念。process是在go语言上的表现就是 goroutine 是实际并发执行的实体，每个实体之间是通过channel通讯来实现数据共享。

Go语言的CSP模型是由协程Goroutine与通道Channel实现：

- Go协程goroutine: 是一种轻量线程，它不是操作系统的线程，而是将一个操作系统线程分段使用，通过调度器实现协作式调度。是一种绿色线程，微线程，它与Coroutine协程也有区别，能够在发现堵塞后启动新的微线程。
- 通道channel: 类似Unix的Pipe，用于协程之间通讯和同步。协程之间虽然解耦，但是它们和Channel有着耦合。

#### 4.5.3、Channel

Goroutine 和 channel 是 Go 语言并发编程的 两大基石。Goroutine 用于执行并发任务，channel 用于 goroutine 之间的同步、通信。

Channel 在 gouroutine 间架起了一条管道，在管道里传输数据，实现 gouroutine 间的通信；由于它是线程安全的，所以用起来非常方便；channel 还提供 “先进先出” 的特性；它还能影响 goroutine 的阻塞和唤醒。

相信大家一定见过一句话：

**Do not communicate by sharing memory; instead, share memory by communicating.**

不要通过共享内存来通信，而要通过通信来实现内存共享。

这就是 Go 的并发哲学，它依赖 CSP 模型，基于 channel 实现。

**channel 实现 CSP**

Channel 是 Go 语言中一个非常重要的类型，是 Go 里的第一对象。通过 channel，Go 实现了通过通信来实现内存共享。Channel 是在多个 goroutine 之间传递数据和同步的重要手段。

使用原子函数、读写锁可以保证资源的共享访问安全，但使用 channel 更优雅。

channel 字面意义是 “通道”，类似于 Linux 中的管道。声明 channel 的语法如下：

```go
chan T // 声明一个双向通道
chan<- T // 声明一个只能用于发送的通道
<-chan T // 声明一个只能用于接收的通道
```

单向通道的声明，用 `<-` 来表示，它指明通道的方向。你只要明白，代码的书写顺序是从左到右就马上能掌握通道的方向是怎样的。

因为 channel 是一个引用类型，所以在它被初始化之前，它的值是 nil，channel 使用 make 函数进行初始化。可以向它传递一个 int 值，代表 channel 缓冲区的大小（容量），构造出来的是一个缓冲型的 channel；不传或传 0 的，构造的就是一个非缓冲型的 channel。

两者有一些差别：非缓冲型 channel 无法缓冲元素，对它的操作一定顺序是 “发送 -> 接收 -> 发送 -> 接收 -> ……”，如果想连续向一个非缓冲 chan 发送 2 个元素，并且没有接收的话，第一次一定会被阻塞；对于缓冲型 channel 的操作，则要 “宽松” 一些，毕竟是带了 “缓冲” 光环。

对 chan 的发送和接收操作都会在编译期间转换成为底层的发送接收函数。

Channel 分为两种：带缓冲、不带缓冲。对不带缓冲的 channel 进行的操作实际上可以看作 “同步模式”，带缓冲的则称为 “异步模式”。

同步模式下，发送方和接收方要同步就绪，只有在两者都 ready 的情况下，数据才能在两者间传输（后面会看到，实际上就是内存拷贝）。否则，任意一方先行进行发送或接收操作，都会被挂起，等待另一方的出现才能被唤醒。

异步模式下，在缓冲槽可用的情况下（有剩余容量），发送和接收操作都可以顺利进行。否则，操作的一方（如写入）同样会被挂起，直到出现相反操作（如接收）才会被唤醒。

小结一下：同步模式下，必须要使发送方和接收方配对，操作才会成功，否则会被阻塞；异步模式下，缓冲槽要有剩余容量，操作才会成功，否则也会被阻塞。

简单来说，CSP 模型由并发执行的实体（线程或者进程或者协程）所组成，实体之间通过发送消息进行通信， 这里发送消息时使用的就是通道，或者叫 channel。

CSP 模型的关键是关注 channel，而不关注发送消息的实体。Go 语言实现了 CSP 部分理论，goroutine 对应 CSP 中并发执行的实体，channel 也就对应着 CSP 中的 channel。

#### 4.5.4、Goroutine

Goroutine 是实际并发执行的实体，它底层是使用协程(coroutine)实现并发，coroutine是一种运行在用户态的用户线程，类似于 greenthread，go底层选择使用coroutine的出发点是因为，它具有以下特点：

- 用户空间 避免了内核态和用户态的切换导致的成本
- 可以由语言和框架层进行调度
- 更小的栈空间允许创建大量的实例

可以看到第二条 用户空间线程的调度不是由操作系统来完成的，像在java 1.3中使用的greenthread的是由JVM统一调度的(后java已经改为内核线程)，还有在ruby中的fiber(半协程) 是需要在重新中自己进行调度的，而goroutine是在golang层面提供了调度器，并且对网络IO库进行了封装，屏蔽了复杂的细节，对外提供统一的语法关键字支持，简化了并发程序编写的成本。

#### 4.5.5、Goroutine 调度器

Go并发调度: G-P-M模型

，channel 使用 make 函数进行初始化。可以向它传递一个 int 值，代表 channel 缓冲区的大小（容量），构造出来的是一个缓冲型的 channel；不传或传 0 的，构造的就是一个非缓冲型的 channel。

两者有一些差别：非缓冲型 channel 无法缓冲元素，对它的操作一定顺序是 “发送 -> 接收 -> 发送 -> 接收 -> ……”，如果想连续向一个非缓冲 chan 发送 2 个元素，并且没有接收的话，第一次一定会被阻塞；对于缓冲型 channel 的操作，则要 “宽松” 一些，毕竟是带了 “缓冲” 光环。

对 chan 的发送和接收操作都会在编译期间转换成为底层的发送接收函数。

Channel 分为两种：带缓冲、不带缓冲。对不带缓冲的 channel 进行的操作实际上可以看作 “同步模式”，带缓冲的则称为 “异步模式”。

同步模式下，发送方和接收方要同步就绪，只有在两者都 ready 的情况下，数据才能在两者间传输（后面会看到，实际上就是内存拷贝）。否则，任意一方先行进行发送或接收操作，都会被挂起，等待另一方的出现才能被唤醒。

异步模式下，在缓冲槽可用的情况下（有剩余容量），发送和接收操作都可以顺利进行。否则，操作的一方（如写入）同样会被挂起，直到出现相反操作（如接收）才会被唤醒。

小结一下：同步模式下，必须要使发送方和接收方配对，操作才会成功，否则会被阻塞；异步模式下，缓冲槽要有剩余容量，操作才会成功，否则也会被阻塞。

简单来说，CSP 模型由并发执行的实体（线程或者进程或者协程）所组成，实体之间通过发送消息进行通信， 这里发送消息时使用的就是通道，或者叫 channel。

CSP 模型的关键是关注 channel，而不关注发送消息的实体。Go 语言实现了 CSP 部分理论，goroutine 对应 CSP 中并发执行的实体，channel 也就对应着 CSP 中的 channel。

#### 4.5.4、Goroutine

Goroutine 是实际并发执行的实体，它底层是使用协程(coroutine)实现并发，coroutine是一种运行在用户态的用户线程，类似于 greenthread，go底层选择使用coroutine的出发点是因为，它具有以下特点：

- 用户空间 避免了内核态和用户态的切换导致的成本
- 可以由语言和框架层进行调度
- 更小的栈空间允许创建大量的实例

可以看到第二条 用户空间线程的调度不是由操作系统来完成的，像在java 1.3中使用的greenthread的是由JVM统一调度的(后java已经改为内核线程)，还有在ruby中的fiber(半协程) 是需要在重新中自己进行调度的，而goroutine是在golang层面提供了调度器，并且对网络IO库进行了封装，屏蔽了复杂的细节，对外提供统一的语法关键字支持，简化了并发程序编写的成本。

#### 4.5.5、Goroutine 调度器

Go并发调度: G-P-M模型

在操作系统提供的内核线程之上，Go搭建了一个特有的两级线程模型。goroutine机制实现了M : N的线程模型，goroutine机制是协程（coroutine）的一种实现，golang内置的调度器，可以让多核CPU中每个CPU执行一个协程。
