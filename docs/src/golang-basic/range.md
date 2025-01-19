---
title: for i,v:=range切片，v地址会变吗？
shortTitle: 4.for...range新版本特性
description: for i,v:=range切片，v地址会变吗？1.22版本新特性？
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-03-09
star: true
order: -4
---
## 1.for i,v:=range切片，v地址会变吗？
1.22之后，在Golang中range遍历切片v的地址是会变的，但是这个v是临时变量，打印的地址不是原数组的地址。

```go
	sc := []int{1, 2, 3, 4, 5}
	for _, v := range sc {
		fmt.Println(&v)
	}
	fmt.Println("----")
	for i := 0; i < len(sc); i++ {
		fmt.Println(&sc[i])
	}
```
输出结果：
```
0xc00000a0d8
0xc00000a110
0xc00000a118
0xc00000a120
0xc00000a128
----
0xc00000e3f0
0xc00000e3f8
0xc00000e400
0xc00000e408
0xc00000e410
```
可见原切片和临时变量v的地址不同，切片中相邻元素地址可能不连续是因为在Go语言中由于内存对齐、内存池分配、垃圾回收等造成的。

在1.22之前for range遍历切片地址是不变的，就是临时变量v的地址。

```go
	sc := []int{1, 2, 3, 4, 5}
	var a []*int
	for _, v := range sc {
		fmt.Println(&v)
		a = append(a, &v)
	}
	fmt.Println(*a[0], *a[1], *a[2], *a[3])
```
这个可以自己测试一下，我这个版本是go version go1.23.0 windows/amd64，如果是1.22之前的可以试一试。
## 2.介绍一些forrange踩坑点。
### 2.1.使用迭代变量时的闭包问题
```go
	funcs := make([]func(), 0, 10)
	for i := 0; i < 5; i++ {
		funcs = append(funcs, func() {
			fmt.Println(i)
		})
	}

	for _, f := range funcs {
		f()
	}
```
输出结果：
```
0
1
2
3
4
```
在这里并没有什么问题，但是在1.22版本前的，可能出现问题输出结果都是6，因为在之前匿名函数中使用循环中的i是引用，所以在forrange时全部打印最后的值，但是在1.22之后这个就修改了，目前正常输出。
### 2.2.修改切片中的元素
```go
// 错误代码
	sc := []int{0, 1, 2, 3, 4}
	for _, v := range sc {
		v = 20
	}
	fmt.Println(sc)
```
以上使用直接提示不可以这样使用，如果要想修改切片某个元素，使用索引来修改。

### 2.3.遍历字典无序
字典forrange遍历无序。

如果我想让字典遍历有序呢？

那就让key排序，然后在依次输出数据。

```go
	mp := map[string]string{"b": "你好", "a": "你们好", "c": "大家都好"}
	keys := make([]string, 0, len(mp))
	for k := range mp {
		keys = append(keys, k)
	}
	sort.Strings(keys)
	for _, i := range keys {
		fmt.Println(i, mp[i])
	}
```
### 2.4.字符串遍历
```go
	s := "你们好啊"
	for i, v := range s {
		fmt.Println(i, string(v))
	}
```
输出结果：
```
0 你
3 们
6 好
9 啊
```
for range遍历字符串，每次迭代返回Unicode代码点，而不是字节。遍历字符串建议用常规for循环。

