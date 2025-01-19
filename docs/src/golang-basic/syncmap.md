---
title: Go sync.map 和原生 map 谁的性能好，为什么？
shortTitle: 7.map和sync.Map的区别？
description: sync.Map的底层结构及实现原理，map和sync.Map的区别？sync.Map的底层原理？
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-04-01
star: true
order: -5
---

## 前言：sync.map和原生map谁的性能好，为什么？

sync.Map在高并发读多写少的场景中性能较好，因为它内置了优化的同步机制（如分段锁），适用于并发访问较高的情况。相比之下，原生map在没有并发操作时性能最好，因为它没有加锁机制，操作更快。但在多线程并发读写时，原生map可能导致数据竞争，因此在并发环境中使用时性能较差。

## 1.sync.Map的底层结构及实现原理

sync.Map是一个结构体：

```go
type Map struct {
	mu Mutex
  	
	read atomic.Value // 后面是readOnly结构体，依靠map实现，仅仅只用来读

	dirty map[interface{}]*entry // 这个map主要用来写的，部分时候也承担读的能力

	misses int // 记录读或删dirty map的次数
}
```
readOnly结构体(是只读结构体)：

```go
type readOnly struct {
	m       map[interface{}]*entry  // read map 读操作所对应的map
  
	amended bool // false 表示数据是完整的
}
```
真正存储数据的地址，其实是在entry结构体中的：

```go
type entry struct {
	p unsafe.Pointer // *interface{}
}
```


### 1.1 双向数据传递（read和dirty的数据流转）
#### dirty map -> read map

 - map结构体里的misses次数达到了len(dirty map)的时候，意味着，很多次都需要在dirty map中查找key，访问dirty map时需要进行锁操作，大大降低了性能。因此就将dirty map提升为read map。
 - Range遍历的时候，如果发现dirty map中有些key在read map中没有，那么就提升为read map。

dirty map转换为read map的过程：

 1. dirty map覆盖 read map
 2. dirty map清零为nil
 3. misses置为0

#### read map -> dirty map

我们知道dirty map->read map后，dirty map已经重置了，所以有新的写请求到来，那么就需要初始化dirty map。就是将read map里非逻辑删除的数据拷贝到dirty map。

> 为什么read map和dirty map要数据流转呢？

如果dirty map不重置为nil，那么就会造成read map和dirty map使用同一个底层map，这样无法隔离数据。所以**read map和dirty map是不同的map，但是entry是共享的**。


### 1.2 entry的nil状态和expunged状态

在sync.Map中，read map和dirty map各自对应的map结构都是一个entry结构体：

```go
// expunged表示硬删除态
var expunged = unsafe.Pointer(new(interface{}))

type entry struct {
   p atomic.Pointer[any]
}
```

在sync.Map中，read map和dirty map中**相同的key的entry都指向了相同的内容**(共享)。所以就不需要维护两份相同的value。

当数据同时在read map和dirty map中，尽量在read map进行删除操作，通过CAS将p的值变为nil，这样在read map和dirty map 中 key 就已经被逻辑删除了。之所以有nil状态就是为了下次对这个key写操作的时候，实现内存复用，提高效率。

当我们将read map上的数据拷贝到dirty map上的时候，已经逻辑删除的数据不会拷贝过去，nil态的entry全部标记为expunged。

那么下次写操作的时候，如何判断是更新read map还是写到dirty map？
如果只有nil这一种标识的话，那么对这个key写的时候就会丢失dirty map的那一份数据，所以我们需要一个新的标识，表示read map和dirty map是否在共享这个key-entry，那就是expunged。所以的那个read map中的标识为expunged时，如果要恢复就需要加锁写到dirty map上。

> entry状态存在的意义

在 sync.Map 中，entry 的状态存在是为了优化并发访问的性能。当删除一个键时，如果该键存在于 read map 中，sync.Map 并不会立即从底层数据结构中移除它，而是将 entry 的状态标记为 nil 或 expunged。这样做的好处是可以避免在高并发场景下频繁的删除和插入操作，减少锁竞争。同时，这些标记也能帮助在将来进行垃圾回收或延迟清理时处理已删除的键值对。

**sync.Map常用的api接口:**

 1. mp.Load(key)
 2. mp.Store(key,value)
 3. mp.Delete(key）

### 1.3 sync.Map的实现原理

 - 通过read和dirty两个字段读写分离，读的数据在read map，写的数据直接写到dirty map
 - 读取时先查询read map，如果未查询到，就查询dirty map，导致misses+1
 - 写入时直接写入dirty map
 - 读取read map不需要加锁，读取dirty map需要加锁
 - misses字段记录dirty map被读取和查询的次数，如果超过len(dirty map)，那么就将数据同步到read map，然后将dirty map置为nil和misses清空

### 1.4 read 和 dirty 的区别?

read 是只读的，并且通过原子操作访问，因此不需要锁，是高效的并发读路径。

dirty 是在写操作或 read 命中失败时才涉及的，用于存储那些还没被同步到 read 中的键值对。访问 dirty 需要加锁保护。



## 2.map压力测试

### 2.1 有无锁和sync.Map的写状态效率测试

```go
package main

import (
	"sync"
	"testing"
)

func BenchmarkAddMapWithUnlock(b *testing.B) {
	for i := 0; i < b.N; i++ {
		m := make(map[int]int)
		for i := 1; i <= 10000000; i++ {
			m[i] = i
		}
	}
}

func BenchmarkAddMapWithLock(b *testing.B) {
	for i := 0; i < b.N; i++ {
		var mx sync.Mutex
		m := make(map[int]int)
		for i := 1; i <= 10000000; i++ {
			mx.Lock()
			m[i] = i
			mx.Unlock()
		}
	}
}
func BenchmarkAddMapWithSyncMap(b *testing.B) {
	for i := 0; i < b.N; i++ {
		var m sync.Map
		for i := 1; i <= 10000000; i++ {
			m.Store(i, i)
		}
	}
}
```

运行结果：

![](https://cdn.golangcode.cn/images/202501181549720.png)

从上面可以看到，在使用map和加锁后map进行写操作的时候都比使用sync.Map的效率高，而且要高出接近4倍。所以sync.Map适合读多写少的情况，写多读少会非常耗时。

## 3.面试题总结

### 3.1、sync.Map的底层原理？

sync.Map采用**空间换时间的取舍策略**以及**实时动态的数据流转策略**，期望使用read map尽量进行读、更新、删除操作用无锁化的操作进行，避免去加锁访问拥有全量数据的dirty map。sync.Map删除还有nil和expunged状态，nil状态可以拦截删除操作在read map，expunged可以正确表示dirty map中没有对应的逻辑删除的key-entry。

sync.Map的底层结构有两个map，通过空间换时间的方式，使用read map和dirty map实现了读写分离。

read map和dirty map之间也有实时的数据流转策略，如果dirty map接收大量的读请求，那么会对dirty map造成巨大压力，所以dirty map就会把所有的数据拷贝给read map，然后将dirty map和misses都重置。当有新的写请求到来，首先初始化dirty map（将read map里不是nil的数据拷贝到dirty map中），同时将nil状态变成expunged状态。

entry的状态有三种nil、expunged、正常。nil状态可以拦截删除操作在read map，expunged是为了标识dirty map中没有read map中的key-entry。nil状态就是优化了对同一个key先删后写的场景。expunged状态代表了dirty map上没有和read map共享一个key-entry，所以在写操作的时候需要加锁写到dirty map。

### 3.2、read map什么时候更新？dirty map 什么时候更新？

当misses大于等于dirty map的时候，将dirty map中的key-entry全部拷贝到read map，然后将dirty map置为nil，misses置为0 。

当dirty map置为nil时，如果有写的请求，那么就初始化dirty map，将非逻辑删除的key-entry拷贝到dirty map，再把nil态修改为expunged态。

### 3.3、read map和dirty map的删除逻辑有什么区别？

read map删除是通过CAS将value设置为nil状态，而dirty map删除是直接删除key-value。read map是延迟删除，dirty mpa是直接删除。

### 3.4、read map和dirty map有什么关联？

read map可以当做dirty map的保护层，使用轻便的原子操作将流量拦截在read map层，防止加锁访问dirty map。

dirty map是当做read map的兜底层，如果在read中没有完成的操作，最终需要加锁，然后尝试在dirty map完成兜底。

如果不断对dirty map进行读删操作，直到misses等于dirty map的大小的时候，需要将dirty map提升到read map，并将dirty map置为nil，清空misses。

当dirty map为nil，会在Store里面触发dirtyLocked流程，将以o(n)的时间复杂度遍历read map，将所有非逻辑删除的key-entry写入到dirty map中，并将entry的nil状态变成expunged。

### 3.5、为什么要设计nil和expunged状态？

设计nil状态是为了标记key-entry已经逻辑删除了，但是key-entry还存在于read map和dirty map中，如果对一个已经删除的key，再进行写，那么就可以直接在read map通过CAS解决。

expunged状态是硬删除态，也是逻辑上的k-v删除，但是key-entry只存在read map中，能正确标识出key-entry是否存在于dirty map中。

## 4.一道算法题(放空一下)

1、假如有一个函数输出2，在这个函数里面使用一个defer，在函数进入的时候输出1，在函数结束的时候输出3 。

```go
package main

import "fmt"

func main() {
	defer funcc()()
	fmt.Println(2)
}

func funcc() func() {
	fmt.Println(1)
	return func() {
		fmt.Println(3)
	}
}
```

## 5.最后

如果以上内容对你有帮助的话也可以关注我的公众号“GolangCode”，获取更多技术干货。

![GolangCode](https://cdn.golangcode.cn/images/202501171944968.png)
