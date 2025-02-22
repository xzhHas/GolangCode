---
title: SET是什么？
shortTitle: 3.SET是什么？
category:
  - Redis
tag:
  - Redis
date: 2024-06-22
---

## 1.SET是什么？
Redis的Set是一个不重复、无序的字符串集合。
## 2.适用场景
适用于无序集合场景，比如某个用户关注了哪些公众号，这些信息就可以放进一个集合，Set提供了查交集、并集的功能，可以很方便地实现共同关注的功能。
## 3.常用操作
- 创建：SADD
- 查询：SISMEMBR、SCARD、SMEMBERS、SSCAM、SINTER、SUNION、SDIFF
- 更新：SADD、SREM
- 删除：DEL

#### 1.写操作

1、SADD，添加元素，返回成功添加了几个元素。




2、SREM，删除元素，返回值为成功删除了几个元素。



#### 2.读操作

1、SISMEMBER，查询元素是否存在。

2、SCARD，查询集合元素个数。可以查看成员个数：SCARD key

3、SMEMBERS，查看集合的所有元素。可以查看所有成员：SMEMBERS set1

4、SSCAN

5、SINTER，返回在第一个集合里，同时在后面所有集合都存在的元素。A交B，交集。

6、SUNION，返回第一个集合里有，且在后续集合中不存在的元素。并集。

## 4.底层实现

Set对象编码方式：INTSET、HASHTABLE。

INSET编码
如果集合元素都是整数，且元素数量不超过512个，就可以用INTSET编码。INTSET编码排列比较紧凑，内存占用少，但是查询
时需要二分查找（INSET下是有序的）。



HASHTABLE
如果不满足INTSET的条件，就需要用HASHTABLE。


## 5.总结（重点）

1、SET编码方式？
SET使用整数集合和字典作为底层编码，当元素都是整数同时元素个数不会超过512个，会使用整数集合编码，否则使用字典编码。

2、SET为什么要用两种编码方式？
SET底层编码是整数集合和字典，当元素数量小于并且全部是整数的时候，会使用整数集合编码，更加节约内存。元素数量变大会使用字典编码，查找元素会更快。
