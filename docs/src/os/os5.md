---
title: 经典同步问题
shortTitle: 5.经典同步问题
date: 2024-09-14
category:
  - 操作系统
tag:
  - 操作系统
---

## 哲学家就餐问题

### 问题引入

现在，有5位哲学家坐在一张圆桌上吃饭，但是只有5个餐具，哲学家有个特殊癖好，就是进餐必须使用两个餐具，进餐后会把餐具放到原来的位置。

那么，如何保证这5位哲学家的进餐有序呢？

### 方案一

学过操作系统的人都知道，进程间的通信方式有很多种，而如何保证进程间的有序执行是一个重要问题。为了避免多个进程同时访问和修改共享内存时发生冲突，可以采用信号量来进行同步控制。

信号量是一种用于进程间同步和互斥的机制，它能有效避免进程间对共享资源的竞争。控制信号量的基本操作就是PV操作。

直接使用PV操作可能会因为多个人同时操作，导致各拿了一个餐具。所以，需要加一把锁控制当前哲学家拿够两把，并进餐完毕，再开锁。

```go
var forks = [5]sync.Mutex{}  // 每个餐具的信号量
var mutex = sync.Mutex{}      // 互斥锁，用于控制哲学家拿餐具的顺序

func philosopher(id int) {
    for {
        think()  // 哲学家思考
        // 尝试获取餐具
        mutex.Lock()          // 进入临界区
        forks[id].Lock()      // 拿左边餐具
        forks[(id+1)%5].Lock()// 拿右边餐具
        mutex.Unlock()        // 离开临界区
        eat()  // 哲学家进餐
        forks[id].Unlock()
        forks[(id+1)%5].Unlock()
    }
}

func main() {
    for i := 0; i < 5; i++ {
        go philosopher(i)  
    }
}
```

### 方案二

如果按照方案一来的话，会导致在相同时间内，只能允许一个哲学家进餐，会浪费剩下的3个餐具。也为了避免都拿1个餐具的情况，我们可以设置**偶数**编号的哲学家**先拿右边的再拿左边的**，**奇数**编号的哲学家**先拿左边的再拿右边的**。

```go
var forks = [5]sync.Mutex{}  // 每个餐具的信号量

func philosopher(id int) {
    for {
        think()  // 哲学家思考
        // 奇数编号的哲学家先拿右边餐具，偶数编号的哲学家先拿左边餐具
        if id%2 == 0 {
            forks[id].Lock()       // 拿左边餐具
            forks[(id+1)%5].Lock() // 拿右边餐具
        } else {
            forks[(id+1)%5].Lock() // 拿右边餐具
            forks[id].Lock()       // 拿左边餐具
        }
        eat()  
        forks[id].Unlock()
        forks[(id+1)%5].Unlock()
    }
}

func main() {
    for i := 0; i < 5; i++ {
        go philosopher(i)  
    }
}
```

### 方案三

其实我们也可以设置三种状态来让哲学家进行进餐，思考状态、进餐状态、饥饿状态。

思考状态：暂时不打算进餐。

进餐状态：正在进餐。

饥饿状态：一旦有进餐结束的，就立马开始进餐。

```go
const (
	// PhilosopherCount 表示哲学家的数量
	PhilosopherCount = 5
)

// State 状态
type State int

const (
	// Thinking 思考状态
	Thinking State = iota
	// Hungry 饥饿状态
	Hungry
	// Eating 进餐状态
	Eating
)

var (
	states  = make([]State, PhilosopherCount)     // 记录哲学家状态
	mutex   sync.Mutex                            // 全局锁保护状态数组
	signals = make([]sync.Cond, PhilosopherCount) // 每个哲学家的信号量
)

// 初始化信号量
func init() {
	for i := 0; i < PhilosopherCount; i++ {
		signals[i].L = &mutex
	}
}

// 测试当前哲学家是否可以进餐
func test(id int) {
	left := (id + PhilosopherCount - 1) % PhilosopherCount
	right := (id + 1) % PhilosopherCount

	if states[id] == Hungry && states[left] != Eating && states[right] != Eating {
		states[id] = Eating
		fmt.Printf("Philosopher %d starts eating.\n", id)
		signals[id].Signal() // 通知该哲学家可以进餐
	}
}

// 哲学家拿起筷子
func pickUp(id int) {
	mutex.Lock()
	defer mutex.Unlock()

	states[id] = Hungry
	fmt.Printf("Philosopher %d is hungry.\n", id)

	// 尝试进入进餐状态
	test(id)

	// 如果无法进餐，则等待
	for states[id] != Eating {
		signals[id].Wait()
	}
}

// 哲学家放下筷子
func putDown(id int) {
	mutex.Lock()
	defer mutex.Unlock()

	states[id] = Thinking
	fmt.Printf("Philosopher %d stops eating and starts thinking.\n", id)

	// 通知左右邻居检查状态
	left := (id + PhilosopherCount - 1) % PhilosopherCount
	right := (id + 1) % PhilosopherCount

	test(left)
	test(right)
}

// 哲学家线程
func philosopher(id int, wg *sync.WaitGroup) {
	defer wg.Done()

	for i := 0; i < 3; i++ { // 每个哲学家尝试进餐 3 次
		time.Sleep(time.Second) // 思考一段时间
		fmt.Printf("哲学家%d正在思考...\n", id)

		pickUp(id)              // 拿起筷子
		time.Sleep(time.Second) // 进餐一段时间
		putDown(id)             // 放下筷子
	}
}

func main() {
	var wg sync.WaitGroup
	wg.Add(PhilosopherCount)

	for i := 0; i < PhilosopherCount; i++ {
		go philosopher(i, &wg)
	}

	wg.Wait()
	fmt.Println("All philosophers have finished eating.")
}
```

