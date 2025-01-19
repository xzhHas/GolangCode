---
title: 内存管理相关内容
shortTitle: 10.逃逸分析
description: 内存管理，逃逸分析，内存模型，内存结构
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-04-13
---
## 1.TCMalloc架构
### 1.1.TCMalloc的架构
TCMalloc三层架构逻辑：
- ThreadCache：线程缓存
- CentralFreeList：中央缓存
- PageHeap：堆内存
### 1.2.堆内存分配

堆内存分配分别有大对象和小对象：

- 小对象：小于等于256kb
- 大对象：大于256kb


#### 小对象的分配过程

1.ThreadCache的alloc充足，则直接分配小对象所需内存；

2.线程缓存mcache的alloc不足，则去中央缓存mcentral获取一个mspan，再分配小对象所需的内存；

3.线程缓存mcache的alloc不足，且中央缓存mcentral不足，则去逻辑处理结构的p.pagecache分配，如果还不足，直接去堆上mheap获取一个mspan，再分配小对象所需内存。


#### 大对象的分配过程

1.逻辑处理结构的pagecache充足，则直接分配大对象所需内存；

2.逻辑处理器结构的pagecache不足，则直接去堆上mheap分配大对象所需内存；

## 2.TCMalloc组件

![](https://cdn.golangcode.cn/images/202501181615141.png)

TCMalloc（Thread-Caching Malloc）是 Google 开发的一种高效的内存分配器，旨在减少多线程环境下的内存分配开销。它通过对内存分配进行优化，来减少锁的竞争和提高内存分配效率。TCMalloc 的主要组件包括：

### 2.1. **Thread-local Cache（线程本地缓存）**
   - 每个线程都拥有自己的本地内存缓存，用于处理小的内存分配请求（通常是 32 KB 以下的内存块）。
   - 通过维护线程私有的缓存来避免频繁的跨线程锁竞争。当一个线程需要分配内存时，它首先会检查自己的本地缓存，从中获取内存，而无需与其他线程竞争全局内存池。
   - 如果线程缓存中没有合适的内存块，则会向中央缓存请求。

### 2.2. **CentralCache**

是所有线程共享的缓存，也是保存的空闲内存块链表，链表的数量与ThreadCache中链表数量相同，当ThreadCache内存块不足时，可以从CentralCache中获取，当ThreadCache内存块多时，可以放回CentralCache。由于CentralCache是共享的，所以他的访问需要加锁。

### 2.3. **Page Heap**

是堆内存的抽象，PageHeap存的也是若干链表，链表保存的是Span，当CentralCache没有内存时，会从PageHeap中获取，把1个Span拆成若干内存块，添加到对应大小的链表中，当CentralCache内存多的时候，会放回PageHeap。

### 2.4. **Span（内存跨度）**
   - TCMalloc 使用 Span 结构来表示连续的内存页。Span 是 Page Heap 中内存管理的基本单元，用于追踪和分配内存块。
   - 一个 Span 可以包含一个或多个页，具体取决于分配的内存大小。较小的分配会占用较少的页数，而较大的分配则会占用更多的页数。

## 参考资料

1、https://blog.csdn.net/weixin_38299404/article/details/126805554

2、https://geektutu.com/post/hpg-escape-analysis.html
