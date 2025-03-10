﻿---
title: 什么是主从模式、哨兵模式？
shortTitle: 12.Redis高可用
category:
  - Redis
tag:
  - Redis
date: 2024-07-27
---

## 1.Redis主从模式、哨兵模式
在前一章持久化文章[【AOF和RDB【Redis持久化篇】 ](https://blog.csdn.net/m0_73337964/article/details/144475425?sharetype=blog&shareId=144475425&sharerefer=APP&sharesource=m0_73337964&sharefrom=link)中，我们知道了Redis是内存数据库，可以通过持久化保存RDB和AOF的方式，将数据持久化到磁盘。如果，这个机器或磁盘出现问题，数据还是没有了，怎么办呢？
### 1.1主从模式
Redis的主从模式（Master-Slave Replication）是一种用于数据复制的机制，使得一个Redis实例（主节点）可以将数据复制到一个或多个Redis实例（从节点）。

就是说，给主节点Master配置一个从节点Slave，当M挂了，S可以顶上。

使用docker实现两个Redis的主从模式。

![[图片]](https://cdn.golangcode.cn/images/202501182052731.png)


那么什么时候，才能发现主节点故障了，需要从节点顶上去？

我们需要对其进行监测，当主节点挂了，自动将项目中Redis的连接配置从主节点修改为从节点，就比如哨兵模式。
### 1.2哨兵模式
Redis 哨兵模式（Sentinel）是一种高可用性解决方案，用于管理 Redis 集群的故障转移和监控。它通过自动检测主节点（Master）故障并自动将一个从节点（Slave）提升为新的主节点来保证 Redis 集群的高可用性。

本质上，哨兵模式就是一个Redis进程，只是这个进程不负责数据存储，它主要是：
1. 监控主从节点是否正常运行。
2. 通过Pub/Sub模式，将故障转移的结果通知给订阅者。
3. 当主节点发生故障时，自动进行故障转移，选择一个最合适的从节点升级为主节点，并通知应用方更新。
那么这里发生故障后，要怎么选择合适的从节点呢？
在 Redis 哨兵模式中，当主节点发生故障时，哨兵集群会选举一个 Leader 哨兵来执行故障转移操作。该 Leader 哨兵的角色是临时的，完成故障转移后，所有哨兵节点将恢复平等的地位，继续工作

**这个新主节点是怎么选择的呢？**

当哨兵集群选出来Leader后，它将选择主从同步的Redis集群中，选择一个节点作为新的主节点，选择策略如下：
1. 过滤故障节点：首先，哨兵会排除已发生故障的节点，确保选择的从节点处于健康状态。
2. 选择优先级最高的从节点：每个从节点都有一个参数replica-priority，它的值决定了该从节点被提升为主节点的优先级。默认值为 100，范围为 0 到 100。哨兵会优先选择replica-priority值最高的从节点。
3. 选择数据同步最完整的从节点：每个从节点都记录了与主节点之间的数据同步进度，即偏移量。偏移量越大，表示该从节点接收到的主节点数据越多，数据同步程度越高。哨兵会优先选择偏移量最大的从节点。如果多个从节点的偏移量相同，则意味着数据同步程度相同。
4. 选择runId最小的从节点：每个 Redis 实例启动时都会生成一个唯一的 runId，用于标识该实例。在多台从节点的偏移量相同的情况下，哨兵会选择 runId 最小的从节点作为新的主节点，因为这个节点的启动时间较早，可能意味着它的稳定性更高。
## 2.Redis集群
2.1集群是什么？

Redis单机能做到写几万，读十万的tps（每秒事务数），大多场景都完全能覆盖。但对于那些特殊场景还是需要更高的tps，就比如秒杀。

2.2怎么访问Redis集群呢？
- 直接连接到单个节点：可以通过指定主节点IP和Port来直接连接Redis集群的某个节点。
- 使用Redis集群代理：Redis集群代理可以将集群所有节点伪装成一个单一的Redis服务器，代理会负责将请求分发给正确的节点，处理节点间的通信，并在出现故障时执行故障转移。

2.3Redis集群用的是一致性Hash算法吗？

不是，用的是哈希槽。一致性Hash算法在服务节点太少时，容易因为节点分布不均匀而造成数据倾斜问题（某一台服务器被大量使用）。
## 3.Redis哈希槽
哈希槽是划分了多个槽位，不同请求会分流到不同的槽位，实现压力均摊。一个Redis Cluster包含16384个哈希槽，存储在Redis Cluster中的所有key都可以通过CRC16算法算出一个Hash值，然后对16384取模，从而找到这个槽。

集群中的每个节点负责管理一定范围的哈希槽。

当客户端请求访问一个key时，Redis会计算该键的哈希槽编号，并将请求转发到负责该哈希槽的节点。如果客户端直接连接到错误的节点，该节点会返回MOVED响应，告知客户端应将请求重定向到正确的节点。
