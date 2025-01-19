---
title: undolog、redolog、binlog到底是什么？
shortTitle: 5.undolog、redolog、binlog
category:
  - MySQL
tag:
  - MySQL
date: 2024-06-12
star: true
---

## 1.什么是undo log、redo log、binlog？

- Undo log（回滚日志）：是存储引擎层生成的日志，实现了事务的原子性，主要用于事务回滚和MVCC。
- Redo log（重做日志）：是存储引擎层生成的日志，实现事务的持久性，主要用于掉电等故障恢复。
- binlog（归档日志）：是Server层生成的日志，主要用于数据备份和主从复制。

## 2.undo log的作用？

如果一个事务在执行过程中，还没有提交事务之前，如果MySQL发生了崩溃，要怎么回滚到事务之前的数据呢？

那么就需要undo log日志，它保证了事务的原子性。

Undo log会先记录更新前的数据，当事务回滚时，可以利用undo log 恢复。

![img](https://cdn.golangcode.cn/images/202501182035277.png)

每当InnoDB对一条记录进行增删改操作时，都需要记录信息到undo log。

与此同时，每一条记录的更新操作产生的undo log日志都有一个roll_pointer指针和一个trx_id事务id。

综上，undo log可以实现事务回滚，保障事务的原子性；实现MVCC。

## 参考资料

[MySQL 日志：undo log、redo log、binlog 有什么用？](https://xiaolincoding.com/mysql/log/how_update.html#为什么需要-undo-log)
