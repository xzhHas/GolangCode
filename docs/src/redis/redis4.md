---
title: Hash、HASHTABLE底层原理
shortTitle: 4.Hash、HASHTABLE底层原理
category:
  - Redis
tag:
  - Redis
date: 2024-06-27
---

## 1.hash是什么？
Redis Hash是一个field、value都为string的hash表，存储在Redis的内存中。
## 2.适用场景
适用于O（1）时间字典查找某个field对应数据的场景，比如任务信息的配置，就可以任务类型为field，任务配置参数为value。
## 3.常用操作
- 创建：HSET、HSETNX
- 查询：HGETALL、HGET、HLEN、HSCAN
- 更新：HSET、HSETNX、HDEL
- 删除：DEL


#### 1.写操作
1、HSET，为集合对应field设置value数据。字段+值。
HSET key field value [field value ...]

![[图片]](https://cdn.golangcode.cn/images/202501182115584.png)

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071637540.png" alt="[图片]" style="zoom:50%;" />

2、HSETNX，如果field不存在，则为集合对应设置value数据。如果存在则不设置。

![[图片]](https://cdn.golangcode.cn/images/202501182115598.png)

3、HDEL，删除指定字段field，可以一次删除多个。
4、DEL，删除Hash对象。
5、HMSET，可以设置多个键值对。在Redis4.0之前，HSET只能设置单个键值对，4.0之后，弃用HMSET，改用HSET。
#### 2.读操作
1、HGETALL，查找全部数据。

![[图片]](https://cdn.golangcode.cn/images/202501182115000.png)

2、HGET，查询field对应的value。
3、HLEN，查找Hash中元素总数。
4、HSCAN，从指定位置查询一定数量的数据。
## 4.原理
Hash底层有两种编码结构，压缩列表和HASHTABLE。同时满足以下两个条件，用压缩列表：
1. Hash对象保存的所有值和键的长度都小于64字节；
2. Hash对象元素个数少于512个。
两个条件任何一条不满足，编码结构就用HASHTABLE。
ZIPLIST其实就是在数据量小的时候将数据紧凑排列，对应到Hash，就是将field-value当做entry放入ZIPLIST。查找key的时间复杂度O(N)。

![[图片]](https://cdn.golangcode.cn/images/202501182115167.png)

HASHTABLE在之前无序集合SET中也有应用，区别就是，在SET中value始终为null，但是Hash中是有对应的值。查找key的时间复杂度O(1)。

![[图片]](https://cdn.golangcode.cn/images/202501182116420.png)

## 5.总结
1、Hash的编码方式是什么？
一个是ZIPLIST，一个是HASHTABLE。ZIPLIST适用于元素较少且单个元素长度较小的情况，其他情况使用HASHTABLE。

2、HASH为什么要用两种编码方式？
采用两种编码方式的原因是ZIPLIST更节约内存，所以在小数据量使用，而数据多时，需要使用HASHTABLE提高更高的查找、更新性能。

## HASTABLE
别，这个模块还没结束呢。学了SET和HASH之后，我们都见到了底层有一个叫HASHTABLE的结构，接下来就去探究一下这是个啥。

#### 1.HASHTABLE简述
简单点说，就是哈希表。那么有什么用呢？
就好比一本书，如果让你一页一页去找是不是很麻烦，要是有一个目录可以直接根据关键字就能定位，是不是效率就更高了。
#### 2.HASHTABLE结构

```c
// redis 5.0.5
typedef struct dictht {
    dictEntry **table;    /* 哈希桶数组，指向实际的hash存储 */
    unsigned long size;   /* 哈希表大小（桶数） */
    unsigned long sizemask; /* 哈希表大小掩码 */
    unsigned long used;   /* 哈希表已使用的桶数量 */
} dictht;
```

![[图片]](https://cdn.golangcode.cn/images/202501182116967.png)

#### 3.渐进式扩容，缩容

```c
// redis 5.0.5
typedef struct dict {
    dictht ht[2];         /* 目前使用的两个哈希表（用于rehash） */
    dictType *type;       /* 数据类型 */
    void *privdata;       /* 私有数据（通常为 NULL），保存需要传给那些类型特定函数的可选参数*/
    long rehashidx;       /* 正在进行的 rehash 操作的桶索引 */
    unsigned long iterators; /* 迭代器数量 */
} dict;
```

为了实现渐进式扩容，redis没有直接把dictht暴露给上层，而是再封装一层，如上。
可以看到dict结构里面，包含了两个dictht结构，也就是两个HASHTABLE结构。dictEntry是链表结构，也就是用拉链法解决哈希冲突，用的头插法。

<img src="https://cdn.golangcode.cn/images/202501182116108.png" alt="[图片]" style="zoom:67%;" />

实际上平时用的都是一个HASHTABLE，在触发扩容之后，就会两个HASHTABLE同时使用，以下是详细流程：
1. 首先，为新Hash表ht[1]分配空间。新表大小为第一个大于等于原表2倍used(已使用的桶数量)的2次方幂。然后迁移ht[0]数据到ht[1]。在ReHash（是指重新计算键的哈希值和索引值）进行期间，每次对字典执行增删改查操作，程序会顺带迁移当前rehashidx在ht[0]上对应的数据，并更新偏移索引。同时，部分情况周期函数也会进行迁移。（这里解释一下这个rehashidx是什么意思：字典同时是拥有ht[0]和ht[1]，将rehashidx设置为0，表示rehash开始；在rehash期间，每次对字典crud，会顺带将ht[0]哈希表在rehashidx索引上的所有kv rehash到ht[1]，当rehash完成后rehashidx+1；随着字典不断操作，最终ht[0]所有键值都会被rehash到ht[1]，这时将rehashidx设置为-1，表示操作结束。注意：在渐进式rehash的过程，如果有crud，如果index大于rehashidx，访问ht[0]，否则访问ht[1]。）
2. 然后，随着字典不断执行，最终在某个时间点上，ht[0]的所有键值都会被Rehash至ht[1]，此时再将ht[1]和ht[0]指针对象互换，同时把偏移索引rehashidx的值设为-1，表示Rehash已完成。
既然知道了扩容的流程，那么扩容时机是什么时候呢？
redis会根据负载因子的情况来扩容：
1. 负载因子大于等于1，说明此时空间已经非常紧张。
2. 负载因子大于5，此时即使有复制命令，也要进行Rehash扩容。
负载因子：k=ht[0].used / ht[0].size
如果扩容太大，但是数据已经减少了，就需要进行缩容，缩容也是渐进式的。那么什么时机缩容呢？
当负载因子小于0.1，即负载率小于10%，此时进行缩容，新表大小为第一个等于原表used的2次方幂。

**总之，ZIPLIST、HASHTABLE面试超级热点，不仅学习这些大致思路，还要掌握一些细节。**
