---
title: INT、EMBSTR、RAW的底层实现
shortTitle: 1.String的底层原理
category:
  - Redis
tag:
  - Redis
date: 2024-06-17
---

## 1.String是什么？
String就是字符串，最大为512MB。
## 2.String怎么用？
适用存储字节数据、文本数据、序列化后的对象数据等。

缓存场景，Value存Json字符串等信息。

计数场景，因为Redis处理命令是单线程，所以执行命令的过程时原子的。因此String数据类型适合计数场景，比如计算访问次数、点赞、转发、库存数量等。

## 3.常用操作
创建、查询、更新、删除。
1. SET 写操作（创建、更新）
SET key value [EX seconds] [PX milliseconds] [NX|XX]
参数：
- EX second：设置键的过期时间为多少秒。
- PX millisecond：设置键的过期时间为多少毫秒。
- NX：只在键不存在时，才对键进行设置操作。SET key value NX等同于SETNX key value。
- XX：只在键存在时，才对键操作。
示例：
SET user:1 "Alice" EX 60  # 设置并在 60 秒后过期
SET counter 100 NX  # 仅在 counter 不存在时设置值

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182054888.png)

2. GET 读操作
GET key
示例：
GET user:1  # 获取 user:1 的值
3. MGET 读操作
MGET key1 key2 ... keyN
示例：
MGET user:1 user:2 user:3  # 获取多个键的值
4. SETNX 写操作
SETNX key value
示例：
SETNX user:1 "Alice"  # 只有在 user:1 不存在时设置其值
5. SETEX 写操作
SETEX key seconds value
- 设置 key 的值并设置过期时间（秒）。
示例：
SETEX session:12345 3600 "user_token"  # 设置并在 3600 秒后过期
## 4.底层实现？
String有三种编码方式：
- INT编码：就是存一个整型，可以用long表示的整数就以这种编码存储。
- EMBSTR编码：如果字符串小于等于阈值字节，使用EMBSTR编码。
- RAW编码：如果字符串大于阈值字节，则用RAW编码。
redis3.0-4.0阈值是39字节，redis5.0是44字节。

EMBSTR和RAW都是由redisObject和SDS两个结构组成，它们的差异在于，EMBSTR下redisObject和SDS是连续的内存，RAW编码下redisObject和SDS内存是分开的。

EMBSTR优点是redisObject和SDS两个结构可以一次性分配空间，缺点在于如果重新分配空间，整体都需要再分配，所以EMBSTR设计为只读，任何写操作之后EMBSTR都会变成RAW，理念是发生过修改的字符串通常会认为是易变的。

我们注意到，EMBSTR和RAW里都有一个叫SDS的结构，那么它是什么呢？

![[图片]](https://cdn.golangcode.cn/images/202501182054120.png)

1.增加长度字段len，快速返回长度；
2.增加空余空间alloc-len，为后续追加数据留余地；
3.不再以'\0'作为判断标准，二进制安全。
## 5.总结（重点）
String可以存储字符数据、文本数据、二进制数据。

### 1、Redis字符串是怎么实现的？

对于Redis字符串对象的创建，有三种编码方式，分别是INT、EMBSTR、RAW，当创建字符串的文本为整数的时候，就是INT编码，如果当创建的字符串大小 小于等于44字节的时候，会使用EMBSTR编码，大于44字节则是RAW编码，但是44字节这个阈值只适用于Redis3.2以及以后的版本，在3.2之前是3.9版本。

### 2、为什么阈值是44字节？

在Redis中是采用jemalloc作为内存分配器的，Redis以64字节为阈值区分大小字符串。Redis对象占用的内存大小由RedisObject和SDS组成，RedisObject16字节，SDS中已分配、已申请、标记三个字段固定占3个字节'\0'占一个，所以能存放的数据就是44字节。

### 3、你知道为什么曾经是阈值是39吗？

3.2之后的版本，SDS结构进行了拆分，EMBSTR用的是sdshdr8，总容量和已使用容量字段减少了6个字节，但由于增加了一个flags字段，所以最终节约了5个字节。

### 4、你知道EMBSTR和RAWEMBSTR的区别吗？

1. EMBSTR只需要一次malloc，而RAWStr需要两次，分配RedsiObject和SDS，同样前者需要一次free后者两次free。
2. EMBSTR读取性能更好，内存碎片率更低。
3. 如果修改EMBSTR（append操作），那么会将EMBSTR转换成RAWString（重新分配内存）。
