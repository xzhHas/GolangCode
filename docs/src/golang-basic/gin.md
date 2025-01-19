---
title: Golang Gin框架快速入门
shortTitle: 13.RESTful架构的go-web框架
description: Golang Gin框架快速入门，RESTful架构的go-web框架
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-04-23
---

## 环境准备

在开始之前，请确保你已经安装了Go语言开发环境。如果还没有安装，请前往[Go官方网站](https://golang.org/dl/)下载并安装。

## 第一步：安装Gin框架

在项目目录下，使用`go get`命令安装Gin框架：

```bash
go get -u github.com/gin-gonic/gin
```

## 第二步：创建项目目录结构

创建一个新的项目目录结构，类似于以下结构：

```
myginapp/
├── main.go
└── go.mod
```

## 第三步：初始化Go模块

在项目根目录下执行以下命令，初始化Go模块：

```bash
go mod init myginapp
```

## 第四步：编写主程序

在`main.go`文件中编写以下代码：

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	// 创建一个默认的路由引擎
	r := gin.Default()

	// 注册一个GET请求的路由和处理函数
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})

	// 启动HTTP服务，监听端口8080
	r.Run(":8080")
}
```

## 第五步：运行项目

在项目根目录下执行以下命令运行项目：

```bash
go run main.go
```

此时，Gin服务已经启动，你可以在浏览器中访问`http://localhost:8080/ping`，应看到以下JSON响应：

```json
{
    "message": "pong"
}
```

## 路由与请求处理

### 路由参数

Gin支持动态路由参数，例如：

```go
r.GET("/user/:name", func(c *gin.Context) {
	name := c.Param("name")
	c.String(http.StatusOK, "Hello %s", name)
})
```

访问`http://localhost:8080/user/John`，会返回`Hello John`。

### 查询参数

获取URL查询参数：

```go
r.GET("/welcome", func(c *gin.Context) {
	firstname := c.DefaultQuery("firstname", "Guest")
	lastname := c.Query("lastname") // 此方法等同于 c.Request.URL.Query().Get("lastname")

	c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
})
```

访问`http://localhost:8080/welcome?firstname=Jane&lastname=Doe`，会返回`Hello Jane Doe`。

### 表单参数

处理表单提交的数据：

```go
r.POST("/form", func(c *gin.Context) {
	message := c.PostForm("message")
	nick := c.DefaultPostForm("nick", "anonymous")

	c.JSON(http.StatusOK, gin.H{
		"status":  "posted",
		"message": message,
		"nick":    nick,
	})
})
```

## 中间件

Gin支持中间件，可以在请求处理前后执行自定义逻辑。例如，编写一个简单的日志中间件：

```go
func Logger() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 开始时间
		t := time.Now()

		// 设置example变量到上下文
		c.Set("example", "12345")

		// 请求前
		c.Next()

		// 请求后
		latency := time.Since(t)
		log.Print(latency)

		// 获取发送的status
		status := c.Writer.Status()
		log.Println(status)
	}
}

func main() {
	r := gin.New()

	// 使用Logger中间件
	r.Use(Logger())

	r.GET("/ping", func(c *gin.Context) {
		example := c.MustGet("example").(string)
		log.Println(example)

		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})

	r.Run(":8080")
}
```

## 处理静态文件

Gin可以方便地处理静态文件，例如HTML、CSS、JavaScript文件等：

```go
r.Static("/assets", "./assets")
r.StaticFS("/more_static", http.Dir("my_file_system"))
r.StaticFile("/favicon.ico", "./resources/favicon.ico")
```

## 模板渲染

Gin支持HTML模板渲染，首先需要加载模板文件，然后在处理函数中渲染模板：

```go
r.LoadHTMLGlob("templates/*")

r.GET("/index", func(c *gin.Context) {
	c.HTML(http.StatusOK, "index.tmpl", gin.H{
		"title": "Main website",
	})
})
```

在`templates`目录下创建`index.tmpl`文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ .title }}</title>
</head>
<body>
    <h1>{{ .title }}</h1>
    <p>Welcome to the Gin framework!</p>
</body>
</html>
```

访问`http://localhost:8080/index`，会渲染并显示模板内容。

