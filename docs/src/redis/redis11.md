---
title: Redis事务和消息队列
shortTitle: 11.Redis事务和消息队列
category:
  - Redis
tag:
  - Redis
date: 2024-07-14
---

## Redis事务和消息队列

<img src="https://cdn.golangcode.cn/images/202501182052315.png" alt="[图片]" style="zoom:80%;" />

MULTI：标记事务开始，后续命令会排队等待执行。
EXEC：提交并执行在事务中的所有命令。
DISCARD：取消事务，丢弃所有队列中的命令。
WATCH：监视一个或多个键，在事务执行前，如果这些键被修改，事务会被中断。

**消息队列**

消息队列是什么？

消息队列有先入先出的特征，一般用于异步流程、消息分发、流量削峰等问题，可以通过消息队列实现高性能、高可用、高扩展的架构。

在Redis中，也有3种方案可以来做一个轻量级的消息队列。（比较出名的消息队列：ActiveMQ、RabbitMQ、Kafka、RocketMQ）

> 方案一：List对象做队列

<img src="https://i-blog.csdnimg.cn/direct/540a39284db24336aa323fb2429ce88c.png" alt="img" style="zoom:67%;" />

List本身是一个双端列表，也是可以支持先入先出的。可以用RPUSH往队列末尾增加元素，另一个客户端用LPOP从对头取出元素，实现先入先出。

虽然可以实现基础功能，但是消费者无法知道LPOP的时机，只能不停按时间间隔轮询，这将是一个巨大负担，所以Redis提供阻塞的POP命令：BRPOP、BLPOP。

<img src="https://i-blog.csdnimg.cn/direct/a341750f57694306807f692ce430ef01.png" alt="img" style="zoom:67%;" />

虽然实现了阻塞式消费了，但是读取消息之后，就出队列了，如果消费失败，消息还得想办法放回去。同时，也不支持多人消费。
> 方案二：Pub/Sub发布订阅模式

<img src="https://cdn.golangcode.cn/images/202501182103869.png" alt="img" style="zoom:67%;" />

当订阅者订阅了某个频道，如果生产者将消息发送到这个频道，订阅者就能收到该消息，这种模式支持多个消费者订阅相同的频道。

<img src="https://i-blog.csdnimg.cn/direct/62ef1831620a4698838cf69f988575cd.png" alt="img" style="zoom:67%;" />

可以看到信息成功收到了，但是也有几个缺点。

1. 没有ACK功能。（消费成功，进行ACK后，才进行下一个信息）
2. 不支持持久化，Redis重启消息会全部丢失。


> 方案三：Stream做消息队列

Redis5.0发布了Stream类型，提供了基本的消息队列的功能。

<img src="https://cdn.golangcode.cn/images/202501182052023.png" alt="方案" style="zoom:80%;" />

