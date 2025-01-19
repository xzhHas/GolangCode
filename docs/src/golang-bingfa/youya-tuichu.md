---
title: 如何在Go语言中优雅地退出Web服务
shortTitle: 2.Go如何优雅地退出Web服务
description: 如何在Go语言中优雅地退出Web服务
author: GolangCode
category:
  - Go
tag:
 - Go
date: 2024-09-01
---

## 1.初始设置

首先，我们使用Gin框架创建一个基本的Web服务。

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	r := gin.Default()
	r.GET("/", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "test"})
	})
}
```

## 2.优雅地处理系统信号

为了使服务能够响应系统中断信号并优雅地关闭，我们需要引入`os/signal`和`context`标准库。我们还将使用`http.Server`而不是Gin的内置方法来更细致地控制HTTP服务。

```go
import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	r := gin.Default()
	r.GET("/", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "test"})
	})

	srv := &http.Server{
		Addr:    ":8081",
		Handler: r,
	}
}
```

## 3.启动服务并监听信号

我们将服务的启动逻辑放在一个goroutine中，这样可以非阻塞地监听操作系统发出的终止信号。

```go
	go func() {
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("listen: %s\n", err)
		}
	}()
```

## 4.信号监听与服务关闭

创建一个信号通道并注册我们希望捕捉的系统信号。当接收到`syscall.SIGINT`或`syscall.SIGTERM`信号时，我们将优雅地关闭服务器。

```go
	quit := make(chan os.Signal)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	if err := srv.Shutdown(ctx); err != nil {
		log.Fatal("Server forced to shutdown: ", err)
	}

	log.Println("Server exiting")
}
```

