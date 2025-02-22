---
title: 什么是ZIPLIST和LINKEDLIST？
shortTitle: 2.List的底层剖析
category:
  - Redis
tag:
  - Redis
date: 2024-06-07
---

## 1.简介

Redis List是一组连接起来的字符串集合。

List最大的元素个数是2的32次方-1，新版本4.0之后是64次方-1。

## 2.使用场景

List作为一个列表存储，属于比较底层的数据结构，比如存储一批任务数据、存储一批消息等。

## 3.常用操作

- 创建：LPUSH、RPUSH
- 查询：LLEN、LRANGE
- 更新：LPUSH、RPUSH、LPOP、RPOP、LREN
- 删除：DEL、UNLINK

### 1.写操作

1、LPUSH从左侧插入，RPSH从右侧插入




2、LPOP移出并获取列表的第一个元素；RPOP移出并获取列表最后一个元素

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182057753.png)

3、LREN key count value，移出值等于value的元素，当count=0，则移出素有等于value的元素，当count>0，则从左到右移出count个，当count<0，则从右到左移出count个。返回被移除元素的数量。

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182057837.png)

4、DEL和UNLINK，DEL是同步删除命令，UNLINK是异步删除命令不会阻塞客户端。

### 2.读操作

1、LLEN，查看list的长度

2、LRANGE，查看从start到stop的元素

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182057632.png)

## 4.底层实现

3.2版本之前，List对象有两种编码方式，ZIPLIST和LINKEDLIST。

ZIPLIST使用条件：

1. 列表对象保存的所有字符串对象长度都小于64字节。
2. 列表对象元素个数少于512个。



LinkedList：



ZIPLIST和LInkedList相比，ZIPLIST内存更加紧凑，所以只有在列表个数或节点数据比较大的时候，才会使用到LINKEDLIST编码。

所以，ZIPLIST是为了在数据较少时节约内存，LInkedList是为了数据多时提高更新效率，而ZIPLIST数据稍多时会导致很多内存复制。

后来，引入了**QUICKLIST**，其实就是ZIPLIST和LInkedLIST的结合体。



原来LInkedList是单个节点，只能存一个数据，现在单个节点存的是一个ZIPLIST，即多个数据。

## 5.压缩列表的优化

平常说的压缩列表一般是指ZIPLIST，一种是LISTPACK 5.0引入的，直到7.0完全替代了ZIPLIST。

### 1.ZIPLIST结构
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182057895.png)

- zlbytes：占用4个字节，记录了整个ziplist占用的总字节数。
- zltail：占用4个字节，指向最后一个entry偏移量，用于快速定位最后一个entry。
- zllen：占用2字节，记录entry总数。
- entry：列表元素。
- zlend：ziplist结束标志，占用1字节，值等于255。

ziplist节点结构

```bash
<prevlen> <encoding> <entry-data>
```

prevlen：表示上一个节点的数据长度。如果前一节点的长度，也就是entry的大小 小于254字节，那么prevlen需要用1字节长的空间来保存这个长度值，255是特殊字符，被zlend占用了。

encoding：编码类型，包含了一个entry的长度信息，可用于正向遍历。

entry-data：实际的数据。

### 2.ziplist更新数据

更新操作可能带来连锁更新。连锁更新是指这个后移，发生了不止一次，而是多次。

什么是连锁更新？

比如增加一个头部新节点，后面依赖它的节点，需要prevlen记录它的大小，原本只用1字节记录，因为更新可能膨胀为5字节，然后这个entry的大小就也膨胀了。所以，当这个新数据插入导致的后移完成之后，还需要逐步迭代更新。



### 3.LISTPACK优化

ziplist需要支持LIST，LIST是双端访问的结构，所以需要能从后向前遍历。

```
<prevlen> <encoding> <entry-data>
```

其中，prevlen就表示上一个节点的数据长度，通过这个字段可以定位上一个节点的数据。

那么，我们可不可以改为不记录这个prevlen，但是又能找到上一个节点的起始位置的办法？

```
<encoding-type> <element-data> <element-tot-len>
```

encoding-type：是编码类型；element-datt：是数据内容；element-tot-len：存储整个节点除它自身的长度。

要找到上一个节点的秘密就需要**element-tot-len**。

element-tot-len所占用的每个字节的第一个bit用于标识是否结束。0是结束，1是继续，剩下7个bit来存储数据大小。

当我们需要找到当前元素的上一个元素时，我们可以从后向前依次查找每个字节，找到上一个Entry的element-tot-len字段的结束标识。

## 6.总结（重点）

### 1、List对象底层编码方式是什么？

3.2版本之前，List底层的编码是ziplist和LInkedlist，当ziplist的节点数量或单个节点大小超过一定阈值时，就会转换成LINKLIST。在3.2版本，结合ziplist和linklist实现了quicklist作为list的新底层编码，而在更新的版本，ziplist优化成了listpack。

### 2、ziplist怎么压缩数据的？

ziplist采用紧凑的内存结构，节点之间的内存空间都是连续的。ziplist的结构大致可以分为三部分，结构头、数据部分、结尾标志；结构头包含了ziplist总字节大小、ziplist节点个数、ziplist最后一个节点的偏移量；数据部分有N个节点组成，单个节点包含节点编码类型，上一个节点的长度，节点实际数据。

### 3、ziplist下list可以从后向前遍历吗？

对于ziplist中的每一个节点，都记录了前一个节点的长度，我们可以用当前遍历节点的首地址减去这个长度，就能找到上一个节点的首地址。

### 4、ziplist的不足

当ziplist的元素个数变得很多之后，查找效率就会下降；而且因为ziplist内存是连续的，当其中一个节点需要更新，或者新增节点时，就需要重新分配内存；再者因为ziplist每一个节点记录了前一个节点的长度，当其中一个节点的长度发生变化时，会导致后面的节点都需要进行更新，引发连锁更新问题。
