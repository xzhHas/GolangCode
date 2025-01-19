---
title: redis是什么、能干什么、主要功能和工作原理的详细讲解
shortTitle: 24.Go如何使用Redis
description: redis是什么、能干什么、主要功能和工作原理的详细讲解
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-05-19
---

## 一、redis是什么、能干什么、主要功能和工作原理的详细讲解

KV数据库

一个开源的内存数据库

**Redis的好处：**

1. 缓存系统，减轻主数据库的压力
2. 计数场景，比如微博，抖音中的关注数和粉丝数
3. 热门排行榜，需要排序的场景特别适合使用ZSET
4. 利用LIST可以实现队列的功能
5. 利用 HyperLogLog 统计UV、PV等数据
6. 使用 geospatial index 进行地理位置相关查询

Redis：https://blog.csdn.net/H1727548/article/details/132512038

## 二、go-redis操作实例

网址：[Go语言操作Redis | 李文周的博客 (liwenzhou.com)](https://www.liwenzhou.com/posts/Go/redis/)

```go
package main

import (
	"context"
	"fmt"
	"time"

	"github.com/redis/go-redis/v9"
)

var rdb *redis.Client

func init() {
	// 连接到 Redis 服务器
	rdb = redis.NewClient(&redis.Options{
		Addr:     "8.130.30.140:6379",
		Password: "", // 如果有密码，请提供密码
		DB:       0,
	})
}

func main() {
	fmt.Println("zsetDemo.")
	zsetDemo()
}

// zsetDemo 操作zset示例
func zsetDemo() {
	// key
	zsetKey := "rank"
	// value
	// 注意：v8版本使用[]*redis.Z；此处为v9版本使用[]redis.Z
	languages := []redis.Z{
		{Score: 90.0, Member: "Golang"},
		{Score: 98.0, Member: "Java"},
		{Score: 95.0, Member: "Python"},
		{Score: 97.0, Member: "JavaScript"},
		{Score: 99.0, Member: "C/C++"},
	}
	ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
	defer cancel()

	// ZADD
	err := rdb.ZAdd(ctx, zsetKey, languages...).Err()
	if err != nil {
		fmt.Printf("zadd failed, err:%v\n", err)
		return
	}
	fmt.Println("zadd success")

	// 把Golang的分数加10
	newScore, err := rdb.ZIncrBy(ctx, zsetKey, 10.0, "Golang").Result()
	if err != nil {
		fmt.Printf("zincrby failed, err:%v\n", err)
		return
	}
	fmt.Printf("Golang's score is %f now.\n", newScore)

	// 取分数最高的3个
	ret := rdb.ZRevRangeWithScores(ctx, zsetKey, 0, 2).Val()
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}

	// 取95~100分的
	op := &redis.ZRangeBy{
		Min: "95",
		Max: "100",
	}
	ret, err = rdb.ZRangeByScoreWithScores(ctx, zsetKey, op).Result()
	if err != nil {
		fmt.Printf("zrangebyscore failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}
}
```

