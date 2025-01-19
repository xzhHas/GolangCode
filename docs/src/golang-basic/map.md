---
title: 为什么 Go map 和 slice 是非线程安全的？
shortTitle: 6.map的底层原理
description: 为什么 Go map 和 slice 是非线程安全的？map的底层原理？
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-03-25
star: true
order: -5
---

## 前言：先来说一下为什么map和slice是非线程安全的？

在 Go 语言中，map和 slice 是非线程安全的，因为它们的底层实现没有针对并发访问做同步保护。对于 map，并发读写可能导致哈希表结构损坏；对于 slice，多个 goroutine 同时修改或扩展底层数组可能导致数据竞争。为了确保线程安全，通常需要使用 sync.Mutex 或 sync.RWMutex 来加锁，或者使用 Go 提供的并发安全数据结构，如 sync.Map。这个sync.Map会在下一节中剖析底层原理，以及解释为什么Go有了map还需要创建一个sync.Map。

直接开始map的底层原理剖析，启动！🚀

## 1.map是什么？

map就是一个kv键值对的集合，可以根据key在o(1)的时间复杂度内取到value。在Golang中map的底层实现，是使用的类似拉链法的方法解决hash冲突的。

### 1.1 什么是hash冲突？

hash表的原理是将多个kv键值对散列的存储到buckets中，buckets是一个连续的数组。存储kv值需要计算hash值和计算索引位置。

1.计算hash值。根据hash函数将key转化为一个hash值。
2.计算索引位置。利用hash值对，桶的数量，取模得到一个索引值，这样就找到了位置。

不妨思考一下，如果我们得到的hash值相同，那么计算得到的hash位置必定相同，这就造成了哈希冲突，这个需要怎么解决？

1.拉链法。
2.开放寻址法。

### 1.2 拉链法


拉链法是一种常见的解决哈希冲突的方法，拉链法主要实现是底层不直接使用连续数组来直接存储数据元素，而是通过数组和链表组合使用，数组里存储一个指针，指向一个链表。如果链表过长，也可以使用优化策略，比如用红黑树代替链表。

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071537835.png" alt="在这里插入图片描述" style="zoom:80%;" />

### 1.3 开放寻址法

开发地址法与拉链法不同，开发地址法是将具体的数据元素存储在数组桶中，在要插入新元素是，先根据哈希函数算出哈希值，根据哈希值计算索引，如果发现冲突了，就从计算出的索引位置向后探测，直到找到未使用的数据槽为止。

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071537150.png" alt="在这里插入图片描述" style="zoom:80%;" />

## 2.map的底层实现原理？

**map的底层实现数据结构实际上就是一个hash表。在运行时表现为指向hmap结构的指针，hmap中记录了桶数组指针，溢出桶指针以及元素个数等字段。每个桶是一个bmap的数据结构体，可以存储8个kv和8个tophash以及指向下一个溢出桶的指针。为了内存紧凑，采用的是先存8个key后再存value。**


### 2.1 map的内存模型
表示map的其实就是hmap结构体：

```go
type hmap struct {
	count     int // # 返回的就是 len(map)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // 溢出桶的bucket近似数
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}
```

B 确定了hmap有2^B次方个桶，每个桶里面都是一个bmap，bmap在编译期间是动态创建的一个新结构：

```go
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}
```

所以，bmap就是我们说的桶。一个桶里面最多可以装8个kv键值对，这些key之所以会在一个桶内，是因为他们经过哈希计算之后，哈希结果是'一类'的。在桶内，会根据hash值高8位决定key放到那个桶内的某个位置上(一个桶8个位置)。

在hmap中有一个extra *mapetra的字段，这个字段的作用是为了防止GC扫描时把溢出桶也处理了，保证GC的扫描性能。
为什么呢？这主要与map的类型有关，如果map的kv是值类型，那么就不用考虑GC扫描，如果是指针类型或着是需要GC扫描的类型，都需要放在extra里面，防止被扫描。

**具体的map底层原理图：**
<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071537024.png" alt="在这里插入图片描述" style="zoom:80%;" />

对于map查找kv键值对的时候，我是这样理解的：**map 底层是一个 hmap 结构，hmap 包含一个 buckets 指针，它指向一个由多个 bmap 组成的数组。哈希值的低 B 位决定使用哪个 bmap，然后根据高位哈希值在该 bmap 内查找具体的键值对。**

### 2.2 map赋值原理

1.在对map进行赋值操作的时候，map一定要先进行初始化，否则panic。

```go
	var m map[int]int
	m[1]=1
```

2.map不是线程安全的，不支持并发读写操作。

```go
func main() {
	m := map[string]int{"a": 1, "b": 2}

	go func() {
		for {
			m["c"] = 3
		}
	}()

	go func() {
		for {
			v, ok := m["a"]
			fmt.Println(v, ok)
		}
	}()
	select {}
}
```
**map的赋值流程：**

1.map写检测，如果为写状态，此时无法进行读取；判断map是否为nil，若为nil，初始化map
2.计算hash值
3.目标桶查找
 - 根据hash值低B位找到桶的位置
 - 判断当前是否处于扩容，若正处于扩容阶段：迁移这个桶，并且还另外多迁移一个桶以及它的溢出桶
 - 获取目标桶的指针，计算出tophash，开始后面的key查找过程

4.key查找

- 判断槽位的tophash和目标tophash

5 key插入，若没有找到这个key，就进行插入操作，**如果位置不够，就创建溢出桶存储**

6 再次判断map的状态，清楚map的写状态


**注意：申请一个新的溢出桶的时候并不会一开始就创建一个溢出桶，因为在map初试化的时候会提前创建好一些溢出桶存储在extra*mapextra字段，当出现溢出现象的时候，这些溢出桶会优先被使用，只有预分配的溢出桶使用完了，才会新建溢出桶。**

### 2.3 map扩容原理

随着不断向map里写入数据，会导致map的数据量很大，hash性能变差，而且溢出桶会越来越多，导致查找的性能很差。所以需要更多的桶时就会触发扩容机制。

map触发扩容机制的两种情况：

 1. map的负载因子大于6.5
 2. 溢出桶的数量过多

map的扩容不是一个原子操作，所以需要!h.growing()判断一下当前的map是否处于扩容状态，避免二次扩容混乱。在这两种扩容情况下，扩容策略是不同的：**负载因子大于6.5，进行双倍扩容，B+1；溢出桶数量过多，等量扩容，一般扩容的溢出桶数量接近正常桶数量。**

**等量扩容**

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071537734.png" alt="在这里插入图片描述" style="zoom:80%;" />

**双倍扩容**

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071537560.png" alt="在这里插入图片描述" style="zoom:80%;" />
需要注意的是，Go语言对map进行扩容的时候，并不是一次性将map的所有数据从旧的桶复制到新的桶，而是在map进行插入、修改、删除key的时候，才会进行迁移。因为map不是线程安全的，所以不能并发读写，只能在写的时候进行数据迁移。

在扩容过程中，要用到hashGrow和growWork两个函数。hashGrow函数只是分配新的buckets，并将老的buckets挂到oldbuckets字段上；growWork函数是进行实际的数据迁移。

其实，map数据迁移时，就是采取写时复制的方式，当有访问bucket时，才会逐渐的将oldbuckets迁移到新的bucket中。

### 2.4 map删除原理


步骤：1.检查h.flag，如果写标位是1，直接panic，因为表示有其他协程同时在进行写操作。 2.计算key的hash，找到桶的位置。如果正在扩容，那就直接触发搬迁操作。  3.找到对应kv位置，进行清零操作，最后count-1 。

找到目标kv删除，将该槽位的tophash设置为emptyOne，如果当前槽位后边是emptyRest，就设置这个槽位也为emptyRest，然后向前检查元素，如果也为emptyOne，就也把tophash设置为emptyRest。这样做的目的是将emptyRest状态尽可能向前推进，这样是为了在查找的时候一旦查到就不需要往后推进了，提高效率。

### 2.5 map的遍历

Golang中map的遍历是无序的。

为什么Golang中的map要用这种随机位置开始遍历呢？

1、因为map的扩容不是一个原子操作，是渐进式的，所以在遍历的时候，可能发生扩容，一旦发生扩容，key的位置就发生了变化，下次遍历map的时候就不可能按原来的顺序了。

2.hash表中数据每次插入的位置是变化的，同一个map变量内，数据删除再添加的位置也有可能变化，因为在同一个桶及溢出链表中的数据位置不分先后。


## 3.map相关题型总结

### 3.1 slice和map分别作为函数参数时有什么区别吗？

slice是一个结构体，做函数参数传递的时候会产生一个结构体的副本，但因为这个结构体里包含了一个指向底层数组的指针，修改底层数组会影响原来的slice，但是需改长度或容量没影响，如果进行了append扩容，那么与原切片就无关了。
map就是指向hamp结构体的指针，所以传参后仍然指向同一个底层数组，所以操作map会影响实参。

### 3.2 map如何顺序读取？

```go
func main() {
	m := map[int]string{11: "a", 2: "b", 3: "c", 0: "d"}

	var ks []int
	for i := range m {
		ks = append(ks, i)
	}
	sort.Ints(ks)

	for _, v := range ks {
		fmt.Println(m[v])
	}
}
```


### 3.3 map使用注意的点，并发安全吗？

注意的点：1、map不是线程安全的，并发读写不安全。
2、map的零值是nil，对nil的读操作是安全的，但是写操作会引发panic。因此，注意使用make前要初始化。
3、key使用多重返回值来检查。
4、key的类型必须是可比较类型。
5、删除一个不存在的key是不会panic。

map不是并发安全的。
### 3.4 map中删除一个key，内存会释放吗？

不会。

map中删除一个key，只是将这个key在底层的tophash、key、value清零，但是内存是没有被释放的，只是修改一个标记，将tophash置为emptyOne，如果后边没有tophash了，就设置当前的tophash为emptyReset，然后向前检查。如果key没有存在的话，也不会panic。

### 3.5 怎么处理对map进行并发访问？有没有其他方案？区别是什么？

1.上锁解决并发安全问题

```go
package main

import (
	"fmt"
	"sync"
)

var mu sync.Mutex

func main() {
	m := make(map[int]string)
	var wg sync.WaitGroup
	wg.Add(2)
	m[0] = "你好"
	go func() {
		defer wg.Done()
		mu.Lock()
		m[0] = "a"
		mu.Unlock()
	}()
	go func() {
		defer wg.Done()
		mu.Lock()
		fmt.Println(m[0])
		mu.Unlock()
	}()
	wg.Wait()
	fmt.Println(m[0])
}
```

2.使用自带的sync.Map进行并发读写

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var m sync.Map
	var wg sync.WaitGroup
	wg.Add(2)
	go func() {
		defer wg.Done()
		m.Store(1, "i")
	}()
	go func() {
		defer wg.Done()
		v, _ := m.Load(1)
		fmt.Println(v)
	}()
	wg.Wait()
	fmt.Println(m.Load(1))
}
```


### 3.6 nil map和空 map有何不同？
nil map是一个未初始化的map，其值为nil。不能向其中添加任何元素，否则会panic。但是可以从nil map中获取元素，这不会引发错误，但总是返回元素类型的零值。

空map是一个已经初始化但没有包含任何元素的map。可以向空map添加元素，也可以从空map中获取元素。

### 3.7 map的数据结构是什么？是怎么实现扩容的呢？


Go 语言中的 `map` 是哈希表的实现，底层结构是通过 `hmap` 和 `bmap` 组合起来的。`map` 的核心结构是 `hmap`，
```go
type hmap struct {
    count     int          // map 中的元素个数
    B         uint8        // bucket 数量的对数，2^B 表示 bucket 的数量
    buckets   unsafe.Pointer // 指向一个数组，每个元素是一个 bucket
    oldbuckets unsafe.Pointer // 指向旧的 bucket 数组，用于扩容过程中的数据搬迁
    ...
}
```
每个 `hmap` 通过 `buckets` 指向一个数组，数组的每个元素是一个 `bucket`，`bucket` 中存储多个键值对，`bucket` 的定义为：
```go
type bmap struct {
    tophash [8]uint8   // 哈希值的高 8 位，用于加快查找
    keys    [8]KeyType // 存储 key
    values  [8]ValueType // 存储 value
}
```
每个 `bucket` 存储 8 个键值对，这使得即使哈希冲突时，查找效率依然较高。

**扩容机制**

map触发扩容的条件是负载因子大于6.5(双倍扩容)或溢出桶数量过多(等量扩容)。触发扩容后进行数据迁移，但是map的数据迁移采用渐进式迁移策略，在每次对map进行写操作的时候，逐步将旧的bucket中的数据迁移到新的bucket，同时顺带处理一部分未迁移旧的bucket。

**总结：map扩容策略就是当负载因子大于6.5时，发生双倍扩容；当溢出桶过多，发生等量扩容。溢出桶过多的标志：溢出桶数量大于正常桶或溢出桶数量大于2^15 。**

### 3.8 map的key为什么是可比较类型？


Go 的 map 是基于哈希表实现的。为了能够将键值对存储在正确的 bucket 中，必须通过对键计算哈希值来确定其存储位置。这要求 map 的键必须是可比较的，以确保：

哈希值稳定：键的哈希值必须是确定且可重复的，只有可比较的类型才能保证键在不同操作中生成相同的哈希值，从而能够正确定位键值对。
冲突处理：当两个键的哈希值相同时，哈希表会使用键的比较来区分它们。如果键不可比较，则无法判断两个键是否相等，也就无法正确处理哈希冲突。

### 3.9 为什么map遍历是无序的？

map在遍历的时候会随机选择一个桶号和槽位开始，因为在map扩容后，会发生key的迁移，原来在同一个bucket中的key，迁移后，有些key的位置就会发生改变。而遍历的过程，就是按顺序遍历bucket。迁移后，key的位置发生了重大变化，就会导致map遍历的结果是无序的了。

### 3.10 为什么map的负载因子是6.5？

```
负载因子 = 哈希表存储的kv数量 / 桶个数
```

负载因子越大，添加的数据就越多，发生hash冲突的几率就更大。

负载因子越小，添加的数据就越少，冲突发生的几率减小，空间浪费比较大，还会提高扩容次数。

每个bucket有8个空位，当负载因子是6.5的时候也就是一个数组桶快用完的时候，所以在此时有必要扩容。

### 3.11 sync.Map和map谁的性能高？

map性能高。

因为sync.Map为了保证线程安全，以空间换时间，采用read和dirty两个map，用了原子操作和锁来实现线程安全，在操作过程加锁会造成性能损耗。而map适配的场景都是简单的，不需要在并发环境下访问，因此不需要锁的操作，这也导致了map是非线程安全的。

## 4.最后

如果以上内容对你有帮助的话也可以关注我的公众号“GolangCode”，获取更多技术干货。

![GolangCode](https://cdn.golangcode.cn/images/202501171944968.png)
