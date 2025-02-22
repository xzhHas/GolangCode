---
title: GMP调度模型的原理
shortTitle: 9.GMP调度模型原理
description: GMP深入理解原理，调度模型，协程，线程，goroutine
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-04-12
star: true
order: -6
---

:::tip

注：个人笔记记录，若有错误尽情提出。部分图片来自刘丹冰老师。

- 原博客地址：[https://learnku.com/articles/41728](https://learnku.com/articles/41728)

:::

## 1.GMP的设计思想


### 1.1 go func(){} 调度流程

<img src="https://cdn.golangcode.cn/images/202501181603591.png" alt="在这里插入图片描述" style="zoom:50%;" />


1、通过go func() 创建一个goroutine；

2、有两个存储G的队列，一个是局部调度器的本地队列、一个是全局队列，新创建的G会先保存在P的本地队列，如果P的本地队列已经满了就会保存到全局队列中；

3、G只能运行在M中，一个M必须持有一个P，M与P是1:1关系，M会从P的本地队列中获得一个可执行状态的G来执行，如果P的本地队列为空，那么就会向其他的MP组合偷取一个可执行的G来执行；

4、一个M调度G执行的过程是一个循环机制；

5、当M执行到某一个G如果发生了syscall或其他阻塞操作，M会阻塞，如果当前有一些G在执行，runtime会把这个线程的M从P中摘除，然后在创建一个新的M（如果有空闲的M就复用空闲的M）来复用这个P；

6、当M系统调用结束，这个G会尝试获取一个空闲的P执行，并放入到这个P的本地队列。如果获取不到P，那么这个M就变成休眠状态，加入到空闲线程，然后这个G就放入全局队列。

> M的自旋状态下获取满足的数量的公式：min(len(globbrunqsize)/GOMAXPROCS+1,len(localrunsize/2))

## 2.GMP的数据结构

### G的数据结构

```go
type g struct {
	stack       stack   // 描述当前goroutine的栈内存范围[stack.lo,stack.hi]
	stackguard0 uintptr // 用于调度器抢占式调度
	stackguard1 uintptr // offset known to liblink
	_panic    *_panic // innermost panic - offset known to liblink
	_defer    *_defer // innermost defer
	m         *m      // 当前 G 占用的 M，可能为空
	sched     gobuf   // 存储 G 的调度相关的数据
	atomicstatus atomic.Uint32// G 的状态
	goid         uint64		  // G 的ID
	waitreason   waitReason // 当状态status==Gwwaiting时等待的原因
	preempt       bool // 抢占信号
	preemptStop   bool // 抢占时将状态修改成 _Gpreempted
	preemptShrink bool // 在同步安全点收缩栈
	lockedm       muintptr// G被锁定只能在这个M上运行
	waiting       *sudog         // 这个G当前正在阻塞的sudog结构体
    ...
}
```

G的主要字段有：

stack：描述了当前goroutine的栈内存范围[stack.lo,stack.hi)；

stackguard0：可以用于调度器抢占式调度；preempt，preemptStop，preemptShrink跟抢占相关；

defer和panic：分别记录这个G最内侧的panic和_defer结构体；

m：记录当前G占用的线程M，可能为空；

atomicstatus：表示G的状态；

sched：存储G的调度相关的数据；

goid：表示G的ID，对开发者不可见；

sched字段的runtime.gobuf结构体：

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181604506.png)


runtime.g的atomicstatus字段存储了当前G的状态：

以下是主要的六种状态：



### M的数据结构




M的字段众多，其重要的有：

g0：Go运行时系统在启动之处创建的，用来调度其他G到M上；

mstartfn：表示M的起始函数，其实就是Go语句携带的那个函数；

curg：存放当前正在运行的G的指针；

p：指向当前与M关联的P；

nextp：用于暂时存放当前M有潜在关联的P；

spinning：表示当前M是否在寻找G，在寻找过程中M处于自旋状态；

lockedg：表示当前与M锁定的那个G，运行时系统会将M和G锁定，一旦锁定就只能相互作用，不接受第三者；

### P的数据结构



最主要的数据结构：

status：表示P的不同状态；

runqhead、runqtail、runq三个字段表示处理器持有的运行队列，是一个长度为256的环形队列，其中存储者等待运行的G列表；

runnext：是线程下一个需要执行的G；

gFree存储P本地的状态为_Gdeed的空闲的G，可重新初始化；

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181605612.png)

### schedt的数据结构



除了这些结构体，也有一些全局变量：

```go
var (
	allm *m				// 所有的 M
	gomaxprocs int32	// P的个数，默认为ncpu的个数
	ncpu 	   int32	
	.....
	sched	   schedt   // schedt 全局结构体
	newprocs   int32
	allplock   mutex    // 全局p队列的锁
	allp       []*p     // 全局p队列，个数为gomaxprocs
)
```

## 3.面试题

### 3.1.什么是GMP？GMP调度流程？（必问）

GMP模型是Go语言中实现并发调度的重要机制。G代表goroutine，M代表操作系统线程，P代表一个调度器，管理调度所需的上下文。G的数量可以很大，不受限于物理线程数；M负责调度工作，主要是将G安排到可用的P上执行；P的数量一般默认为物理CPU核心数，但也可以自己通过GOMAXPROCS。

**调度器的启动**

Go程序的真正启动函数**runtime.rt0_go**：

1.初始化g0和m0，并将二者互相绑定，m0是程序启动后的初始线程，g0是m0的系统栈代表的G结构体，负责普通G在M上的调度切换；

2.schedinit：进行各种运行时组件初始化工作，这包括我们的调度器与内存分配器、回收器的初始化；

3.newproc：负责根据主函数main的入口地址创建可被运行时调度的执行单元；

4.mstart：开始调度器的调度循环；

首先，启动调度器的启动函数**schedinit()**：会设置M的最大数量10000，实际不会达到；会分别调用stackinit()、mallocinit()、gcinit()等执行goroutine栈初始化、进行内存分配器初始化、进行系统线程M0的初始化、进行GC垃圾回收器的初始化；然后将p的个数设置为GOMAXPROCS，最后会调用runtime.procresize()函数初始化P列表。

**schedinit()函数**负责M、P、G的初始化过程。M、P、G彼此的初始化顺序遵循：mcommoninit、procresize、newproc，分别负责初始化M资源池(allm)、P资源池(allp)、G的运行现场(g.sched)以及调度队列(p.runq)。

**mcomoninit()函数**主要负责对m0进行一个初步的初始化，并将其添加到schedt全局结构体中，这里访问schedt会加锁。

**runtime.procresize()函数**的执行过程如下：

1.如果全局变量allp切片中的P数量少于期望数量，会对切片进行扩容；

2.使用new创建新的P结构体并调用runtime.p.init初始化刚刚扩容的P；

3.通过指针将线程m0和处理器allp[0]绑定到一起；

4.通过runtime.p.destroy释放不再使用的P结构；

5.通过切片截断改变全局变量allp的长度，保证它与期望P数量相等；

6.将除allp[0]之外的处理器P全部设置为_Pidle并加入到全局schedt的空闲P队列中；

**runtime.newproce()函数**主要是调用runtime.newproc1()获取新的goroutine结构，将新的G放入P的运行队列，M启动时唤醒新的P执行G。

​	**runtime.newproc1()函数**主要执行：

​	1.获取或者创建新的goroutine结构体，会从处理器gFree列表中查找空闲的goroutine，如果不存在空闲的goroutine，会通过runtime.malg创建一个栈大小足	够的新结构体，新创建的G状态为_Gdead；

​	2.将传入的参数callergp、callerpc、fn更新到G的栈上，初始化G的相关参数；

​	3.将G的状态设置为_Grunnable状态，返回；

​	runtime.newproc1()函数主要调用了**runtime.gfget()**函数获取G：

​    **runtime.gfget()函数**的主要逻辑是：当P的空闲列表gFree为空时，从sched持有的全局空闲列表gFree中移动最多32G到当前的P的空闲列表上，然后从P的	        	gFree列表头返回一个G，如果还是没有，则返回空，说明获取不到空闲的G。

​	在runtime.newproc1()函数中，如果不存在空闲的G，会通过**runtime.malg()**创建一个栈大小足够的新结构体。

回到runtime.newproce()函数，在获取到G后，调用**runtime.runqput()函数**将G放入到P本地队列或全局队列。

**runtime.runqput()函数**的主要逻辑是：

1.保留一定的随机性，设置next为false，即不将当前G设置为P的下一个执行的G；

2.当next为true是，将G设置到P的runnext作为P执行的下一个执行的任务；

3.当next为false并且本地运行队列还有剩余空间时，将goroutine加入处理器持有的本地运行队列；

4.当处理器的本地运行队列已经没有剩余空间时，就会把本地队列中的一部分G和待加入的G通过**runtime.runqputslow**添加到调度器持有的全局运行队列上；

**runtime.runqputslow()函数**会把P本地环形队列的前一半G获取出来，跟传入的G组成一个列表，打乱顺序，再放入全局队列。

**调度循环**

runtime.rt0_go函数在调用runtime.schedinit()初始化了调度器、调用了runtime.newproc()创建了main函数的G后，会调用**runtime.mastrt函数**启动M去执行G。

runtime.mstart()会直接调用runtime.mstart0()函数。

runtime.mstart0()函数主要调用runtime.mstart1()，runtime.mstart1()保存调度信心后，会调用**runtime.schedule()进入调度循环**，寻找一个可执行的G并执行。

**runtime.schedule()函数**会查找待执行的goroutine：

1.当全局队列中有待执行的G时，通过schedtick对61取模，表示每61次会有一次从全局队列中查找对应的G，这样可以避免两个G在P本地队列互换一直占有本地队列；

2.调用runtime.runqget()函数从P本地的运行队列中获取待执行的G；

3.如果前两种都没找到，那么会通过runtime.findrunnable()进行阻塞地查找G；

runtime.schedule函数从全局队列中获取G的函数是**runtime.globrunqget()**：会从全局队列获取n个G，第一个G返回给调度器去执行，剩下的n-1个G放入到本地队列，其中，**n一般为全局队列的长度/P处理器个数+1**，含义是平均每个P应该从全局队列中承担的G数量，且不能超过P本地长度的一半。

runtime.schedul函数从本地队列中获取G的函数是**runtime.runqget()**：本地队列的获取从P的runnext字段获取，如果不为空则直接返回；如果runnext为空，那么从本地环形队列头指针遍历本地队列，取到则返回。

**阻塞式获取G的runtime.findrunnable**看起来很繁琐，其实是按 local->global->netpoll->local->global->netpoll 的顺序获取。

runtime.findrunnable的主要工作：

1.首先检查是否正在进行GC，如果是则休眠当前的M；

2.尝试从本地队列中获取G，如果取到，则直接返回，否则继续从全局队列中找G，如果找到则直接返回；

3.检查netpoll网络轮询器是否有G，如果有，就直接返回；

4.如果此时仍然无法找到G，则从其他P的本地队列中偷取，从其他P的本地队列偷取的工作会执行四轮，如果找到G，则直接返回；

5.所有的可能性尝试过了，在准备休眠M之前，还要进行额外的检查；

6.首先检查此时是否GC mark阶段，如果是，则直接返回mark阶段的G；

7.如果仍然没有，则对当前的P进行快照，准备对调度器进行加锁；

8.当调度器被锁住后，仍然还需要再次检查这段时间里是否有进入GC，如果已经进入了GC，则回到第一步，阻塞M并休眠；

9.如果又在全局队列中发现了G，则直接返回；

10.当调度器被锁住后，我们彻底找不到任务了，则归还释放当前的P，将其放入idle链表中，并解锁调度器；

11.当M、P 已经解绑后，我们需要将M的状态切换出自旋状态，并减少nmspinning；

12.此时仍然需要重新检查所有的队列，如果我们又在全局队列中发现了g，则直接返回；

13.还需要再检查是否存在poll网络的G，如果有，则直接返回；

14.什么也没找到，休眠当前的M；

在 Go 语言的调度器中，`runtime.schedule()` 是负责调度 G（goroutine）的核心函数。它首先通过 `runtime.globrunqget()` 尝试从全局队列获取一个可运行的 G。如果全局队列没有 G，则调用 `runtime.runqget()` 从当前 P（Processor，本地队列）中获取。如果本地队列也没有找到 G，调度器将调用 `runtime.findrunnable()`，从其他地方（例如其他 P 的队列、网络轮询器等）尝试获取一个可执行的 G。

当成功获取到一个可执行的 G 后，调度器将调用 `runtime.execute()`，该函数负责将 G 绑定到 M（OS 线程）上执行。在 `runtime.execute()` 中，会调用 `runtime.gogo()`，将 G 的上下文切换到当前的 M 上，开始真正的执行。

当 G 的执行即将结束时，会进入 `runtime.goexit()` 函数，该函数负责清理 G 的状态，最终调用 `runtime.goexit1()` 和 `runtime.goexit0()` 进行进一步的收尾工作。最后，在 `runtime.goexit0()` 的结尾，调度器会再次调用 `runtime.schedule()`，重新进入下一次的调度循环。

这样，整个调度流程形成了一个循环：从选择 G 到执行 G，再到结束 G，并进入下一次调度。

### 3.2.什么是Workstealing机制？Handoff机制？

Workstealing：M优先执行其绑定的P的本地队列的G，如果本地队列为空，可以从全局队列获取G运行，也可以从其它M绑定的P的运行队列偷取G运行。

Handoff：当M因为G进行系统调用阻塞时，M释放绑定的P，把P转移给其他空闲的M执行。

### 3.3.基于协作式的抢占式调度？基于信号的抢占式调度？

每个真正运行的G，如果不被打断，将会一直运行下去，为了保证公平，防止新创建的G一直获取不到M执行造成饥饿问题，Go程序会保证每个G运行10ms就要让出M，交给其他G去执行。

为了避免长时间占用CPU的goroutine影响程序的并发性，从Go1.14开始引入了抢占式调度，他通过给运行时间过长的goroutine发送抢占信号，让它强制交出CPU以便调度器能够调度其他goroutine，避免长时间运行造成的阻塞。

### 3.4.P的数量有限制吗？M的数量有限制吗？G的数量有限制吗？

P的数量由启动时环境变量GOMAXPROCS或者runtime.GOMAXPROCS()决定，数量不能超过可用的CPU核心数量。

 M的数量设置为10000个，但是达不到。

G的数量受内存和系统资源限制。

### 3.5.进程线程协程有什么区别？

**进程**、**线程**、和**协程**是并发编程中的三个不同概念，它们在操作系统级别和编程模型中的作用不同。下面是它们的区别：

**进程 (Process)**

- **定义**：进程是操作系统中资源分配的基本单位。每个进程都有自己独立的内存空间、文件描述符、堆栈等系统资源，操作系统通过进程进行资源管理和隔离。
- **特点**：
  - **独立性**：进程间彼此独立，内存空间互不干扰。一个进程的崩溃不会影响其他进程。
  - **上下文切换代价大**：因为进程有独立的内存空间，所以进程之间的切换需要保存和恢复大量上下文信息，切换代价相对较高。
  - **进程间通信复杂**：由于不同进程的内存空间隔离，进程间通信（IPC）需要借助操作系统提供的机制，如管道、消息队列、共享内存等。
  - **创建代价大**：进程的创建和销毁需要操作系统进行大量的资源分配和回收操作。

 **线程 (Thread)**

- **定义**：线程是 CPU 调度的基本单位。一个进程可以包含多个线程，线程之间共享进程的内存空间和资源。
- **特点**：
  - **共享资源**：同一进程中的线程共享相同的内存空间、文件句柄等资源，所以线程间通信（例如通过共享内存）非常高效。
  - **轻量级**：线程比进程轻量得多，创建和销毁的代价较小，线程间的上下文切换也比进程快，因为线程共享同一个内存空间，不需要进行复杂的内存操作。
  - **独立执行**：每个线程有自己的栈、寄存器等执行上下文，能够独立调度和执行。
  - **多线程问题**：由于共享内存，线程间通信虽然高效，但容易出现竞争条件、死锁等问题，因此需要小心使用同步机制（如互斥锁、信号量等）来管理资源访问。

 **协程 (Coroutine)**

- **定义**：协程是比线程更轻量的执行单元，也叫作"用户态线程"。协程的调度通常是在用户态完成的，不需要操作系统内核的参与。协程通过主动让出控制权进行调度，通常用于实现非阻塞式并发编程。
- **特点**：
  - **轻量级**：协程的创建和销毁非常廉价，相比于线程，协程不需要操作系统提供上下文切换机制，协程的切换发生在用户态，因此切换代价极低。
  - **非抢占式调度**：协程的调度是协作式的，协程主动放弃 CPU，程序员需要显式地指定何时让出执行权（例如使用 `yield`、`await` 或其他机制）。这减少了并发竞争，但可能导致长时间运行的协程阻塞整个系统。
  - **栈切片**：协程通常使用更小的栈（有的语言会使用分段栈或堆栈切片），因此它可以运行大量协程，而不会像线程那样消耗大量内存。
  - **并发而非并行**：协程通常是并发执行的，但不一定并行执行（即同一时间只有一个协程运行）。多核并行通常需要协程与线程或进程结合使用。
  

**对比总结**

| **特性**         | **进程**                               | **线程**                       | **协程**                                  |
| ---------------- | -------------------------------------- | ------------------------------ | ----------------------------------------- |
| **调度方式**     | 由操作系统内核调度                     | 由操作系统内核调度             | 由用户态调度                              |
| **资源占用**     | 独立资源，占用较大                     | 共享资源，较轻量               | 更轻量，极少的资源占用                    |
| **切换代价**     | 切换代价高（需要切换整个进程的上下文） | 切换代价中等（切换线程上下文） | 切换代价低（用户态内调度）                |
| **并发并行**     | 并发和并行                             | 并发和并行                     | 并发（通常），可结合多线程实现并行        |
| **通信方式**     | 通过 IPC 机制，如管道、消息队列        | 共享内存，通信简单             | 通过共享变量或消息传递，高效              |
| **阻塞**         | 系统调用会阻塞整个进程                 | 系统调用会阻塞整个线程         | 系统调用可避免阻塞，异步非阻塞            |
| **常见应用场景** | 独立应用程序，多任务处理               | 多线程并发任务，如 Web 服务器  | 高并发异步任务处理，如网络 I/O 密集型应用 |

实际应用

- **进程**通常用于需要完全隔离的独立任务，比如浏览器的多个标签页或后台服务。
- **线程**用于同一应用程序中需要并发执行的任务，如 Web 服务器处理多个请求。
- **协程**多用于异步编程和高并发场景，尤其是在 I/O 密集型任务中（如网络编程、数据库操作），如 Go 的 Goroutine、Python 的 `asyncio` 等。



## 最后

GMP这部分难度比较大，更多的笔记和面试题，可以关注我的公众号获取，以上是部分资料。

如果以上内容对你有帮助的话也可以关注我的公众号“GolangCode”，获取更多技术干货。

![GolangCode](https://cdn.golangcode.cn/images/202501171944968.png)
