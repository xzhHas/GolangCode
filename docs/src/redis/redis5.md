---
title: 跳表是什么？
shortTitle: 5.跳表是什么？
category:
  - Redis
tag:
  - Redis
date: 2025-07-02
---

## 1.跳表是什么？
跳表是Redis有序集合ZSet底层的数据结构，跳表在ZSET中尤其重要。
跳表的本质还是链表，只是在普通链表的基础上，增加了多级的索引，通过索引可以一次实现多个节点的跳跃，提高性能。

**跳表的结构**

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071635494.png" alt="[图片]" style="zoom:67%;" />

标准的跳表（Redis不是使用标准的跳表）有以下限制：
1. score值不能重复；
2. 只有向前指针，没有回退指针。

## 2.Redis的跳表实现

<img src="https://cdn.golangcode.cn/images/202501182113490.png" alt="[图片]" style="zoom:67%;" />

<img src="https://cdn.golangcode.cn/images/202501182113416.png" alt="[图片]" style="zoom:67%;" />

**Redis跳表单个节点有几层？**

层次的决定，需要比较随机，Redis是使用概率均衡的思路来确定新插入节点的层数。

Redis跳表决定每一个节点，是否能增加一层的概率为25%，而最大层数限制在Redis5.0是64层，Redis7.0是32层。

**Redis跳表优化了多少？**

O(N)降低到log(N)。
## 总结（重要）
### 1、跳表是什么？和普通链表的区别？

跳表也算链表，不过相对普通链表，增加了多级索引，通过索引可以实现O(logN)的元素查找效率。

### 2、聊聊跳表的查找过程？
从高级索引往后找，如果下个节点比当前大，就降级继续找。

### 3、跳表查询节点总数的平均时间复杂度？
跳变编码模式下，查询节点总数的平均时间复杂度是O(1)，因为跳表头结构中定义了一个保存节点数量的字段Length，源码中调用查询节点总数的api时会直接返回这个字段。

### 4、跳表中一个节点的层高是怎么决定的？
跳表插入新节点会计算一个随机的层高，跳表的每一个节点一开始默认都是1层，然后每增加一层的概率都是25%，在5.0版本最高为64层。

### 5、跳表插入一条数据的平均时间复杂度？
跳表是一种支持多级索引的结构，查询效率媲美二分查找，插入一条数据的时间复杂度为OlogN。

### 6、跳表插入数据会影响其他节点吗？
不会。节点层高在创建时就确认了，不会被新插入节点影响。新插入节点只会影响每一层前一跳、后一跳的关联指针。

## ZSET

### 1.ZSET是什么？

ZSET就是有序集合，也叫SORTED SET，是一组按关联积分有序的字符串集合，这里的分数是个抽象概念，任何指标都可以抽象为分数，以满足不同场景。积分相同的情况下，按字典序排序。
## 2.适用场景
用于需要排序集合的场景，最为典型的就是游戏排行榜。
## 3.常用操作
- 创建：ZADD
- 查询：ZRANGE、ZCOUNT、ZRANK、ZCARD、ZSCORE
- 更新：ZADD、ZREN
- 删除：DEL、UNLINK
1.写操作
**1、ZADD key scoremember [score member ...]** 
向ZSET增加数据，如果key已经存在，则更新对应数据。
扩展参数：
- XX：仅更新存在的成员，不添加新成员。
- NX：不更新存在的成员，只添加新成员。
- LT：更新新的分值比当前分值小的成员，不存在则新增。
- GT：更新新的分值比当前分值大的成员，不存在则新增。

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071636230.png" alt="[图片]" style="zoom: 80%;" />

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071636880.png" alt="[图片]" style="zoom:50%;" />

2、ZREM key member[member ...] ，删除ZSET中的元素。
2.读操作
1、ZCARD key，查看成员总数。
**2、ZRANGE key start stop，查看从start到stop范围的ZSET数据。
3、ZREVRANGE key start stop，从大到小遍历。**
4、ZCOUNT key min max，计算min-max积分范围的成员个数。
5、ZRANK key member，查看ZSET中的member的排名索引。
6、ZSCORE key member，查询ZSET中成员的分数。

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071636089.png" alt="[图片]" style="zoom:80%;" />

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071636592.png" alt="[图片]" style="zoom:67%;" />

## 4.底层实现
ZSET编码有两种方式，一种是ZIPLIST，另一种是SKIPLIST+HASHTABLE。
ZIPLIST编码的使用条件：
1. 列表对象保存的所有字符串对象长度都小于64字节。

2. 列表对象元素个数少于128个。

若有一条不满足，编码就使用SKIPLIST+HASHTABLE。

SKIPLIST是一种可以快速查找的多级链表结构。并且还使用HASHTABLE来配合查询O(1)。

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071636702.png" alt="[图片]" style="zoom:67%;" />

