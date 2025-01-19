---
title: slice的底层原理与实践？切片？数组？
shortTitle: 5.切片和数组的区别
description: slice的底层原理与实践，切片和数组的区别
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-03-11
star: true
order: -3
---


## 1.切片通过函数，传的是什么？

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	s := make([]int, 5, 10)
	printSlice(&s)
	test(s)
}

func test(i []int) {
	printSlice(&i)
}

func printSlice(s *[]int) {
	/*
		reflect.SliceHeader：用来描述切片的底层实现
		unsafe.Pointer：是一个通用类型的指针，可以用作不同类型指针相互转换
	*/
	ss := (*reflect.SliceHeader)(unsafe.Pointer(s))
	fmt.Printf("%+v\n", ss)
}
```
运行结果：

```
&{Data:824633827808 Len:5 Cap:10}
&{Data:824633827808 Len:5 Cap:10}
```

## 2.在函数里面改变切片，函数外的切片会改变吗？

**1.切片做参数传递，修改切片内容影响原切片。**

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	s := make([]int, 5)
	case1(s)
	printSliceStruct(&s)
}

func case1(s []int) {
	s[1] = 1
	printSliceStruct(&s)
}

func printSliceStruct(s *[]int) {
	/*
		reflect.SliceHeader：用来描述切片的底层实现
		unsafe.Pointer：是一个通用类型的指针，可以用作不同类型指针相互转换
	*/
	ss := (*reflect.SliceHeader)(unsafe.Pointer(s))
	fmt.Printf("%+v，%v\n", ss, s)
}
```
输出结果：

```
&{Data:824633779184 Len:5 Cap:5}，&[0 1 0 0 0]
&{Data:824633779184 Len:5 Cap:5}，&[0 1 0 0 0]
```

**2.引用参数传递的时候，在另个函数内部扩容了，那么就会分配新的内存地址，再修改这个切片就不会影响原切片了。**

**具体点可以这样来理解：append切片的时候如果容量还没满，那么这个新切片只是引用原来的底层数组，只不过是长度不一样，可以说是返回的就是原切片；如果容量已满那么扩容后返回的就是一个新切片。**

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	s := make([]int, 5)
	case2(s)
	printSliceStruct(&s)
}

func case2(s []int) {
	s = append(s, 0)
	s[1] = 1
	printSliceStruct(&s)
}

func printSliceStruct(s *[]int) {
	/*
		reflect.SliceHeader：用来描述切片的底层实现
		unsafe.Pointer：是一个通用类型的指针，可以用作不同类型指针相互转换
	*/
	ss := (*reflect.SliceHeader)(unsafe.Pointer(s))
	fmt.Printf("%+v，%v\n", ss, s)
}
```
输出结果：

```
&{Data:824633827808 Len:6 Cap:10}，&[0 1 0 0 0 0]
&{Data:824633779184 Len:5 Cap:5}，&[0 0 0 0 0]
```

**总结：传递切片的时候如果切片的修改要反应外部最好使用指针。就是说，如果传值，那么扩容前可以反应到外部，一旦扩容就不再影响外部，传指针的话，无论是否扩容，指针都能追踪上。**

以下是这个案例：

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	s := make([]int, 5)
	//case1(s)
	//case2(s)
	case3(&s)
	printSliceStruct(&s)
}
func case3(s *[]int) {
	*s = append(*s, 0)
	(*s)[1] = 1
	printSliceStruct(s)
}
func printSliceStruct(s *[]int) {
	/*
		reflect.SliceHeader：用来描述切片的底层实现
		unsafe.Pointer：是一个通用类型的指针，可以用作不同类型指针相互转换
	*/
	ss := (*reflect.SliceHeader)(unsafe.Pointer(s))
	fmt.Printf("%+v，%v\n", ss, s)
}
```
输出结果：

```
&{Data:824634425344 Len:6 Cap:10}，&[0 1 0 0 0 0]
&{Data:824634425344 Len:6 Cap:10}，&[0 1 0 0 0 0]
```

## 3.截取切片
问：通过`:`操作得到的新切片和原切片是什么关系？

答：截取切片只是修改了底层数组的起始指向和末尾指向，长度为截取长度，容量为起始指向到末尾指向的大小。用截取切片初始化新切片也是值传递。

## 4.删除元素

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	s := []int{0, 1, 2, 3, 4}
	printSliceStruct(&s)

	s1 := append(s[:1], s[2:]...)

	printSliceStruct(&s1)
	printSliceStruct(&s)
}

func printSliceStruct(s *[]int) {
	/*
		reflect.SliceHeader：用来描述切片的底层实现
		unsafe.Pointer：是一个通用类型的指针，可以用作不同类型指针相互转换
	*/
	ss := (*reflect.SliceHeader)(unsafe.Pointer(s))
	fmt.Printf("%+v，%v\n", ss, s)
}
```
输出结果：

```
&{Data:824633779184 Len:5 Cap:5}，&[0 1 2 3 4]
&{Data:824633779184 Len:4 Cap:5}，&[0 2 3 4]
&{Data:824633779184 Len:5 Cap:5}，&[0 2 3 4 4]
```

分析：

切片没有内置的删除函数，而是使用append函数实现删除的功能。

第二个print打印和第三个print打印相比之下，可以知道append函数实现的删除不是真的删除，而是后边的数据向前覆盖，这也导致最后的数据没有改变。

## 5.新增元素
Golang中新增元素就是使用append函数，有两种情况：

 1. 切片的底层数组容量足够。那么，使用append添加新元素后的切片与元切片共享底层数组。
 2. 切片的底层数组容量不足。那么，append会返回一个新的底层数组。


```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	case1()
}

func case1() {
	sc := make([]int, 3, 5)
	printSliceStruct(&sc)
	s1 := append(sc, 1)
	printSliceStruct(&sc)
	printSliceStruct(&s1)
	s1[0] = 10
	printSliceStruct(&sc)
	printSliceStruct(&s1)
	fmt.Println("--")
	s2 := append(sc, 1, 2, 3, 4, 5, 6)
	printSliceStruct(&sc)
	printSliceStruct(&s2)
	s2[2] = 20
	printSliceStruct(&sc)
	printSliceStruct(&s1)
}

func printSliceStruct(s *[]int) {
	/*
		reflect.SliceHeader：用来描述切片的底层实现
		unsafe.Pointer：是一个通用类型的指针，可以用作不同类型指针相互转换
	*/
	ss := (*reflect.SliceHeader)(unsafe.Pointer(s))
	fmt.Printf("%+v，%v\n", ss, s)
}
```
输出结果：

```
&{Data:824633779184 Len:3 Cap:5}，&[0 0 0]
&{Data:824633779184 Len:3 Cap:5}，&[0 0 0]
&{Data:824633779184 Len:4 Cap:5}，&[0 0 0 1]
&{Data:824633779184 Len:3 Cap:5}，&[10 0 0]
&{Data:824633779184 Len:4 Cap:5}，&[10 0 0 1]
--
&{Data:824633779184 Len:3 Cap:5}，&[10 0 0]
&{Data:824633827808 Len:9 Cap:10}，&[10 0 0 1 2 3 4 5 6]
&{Data:824633779184 Len:3 Cap:5}，&[10 0 0]
&{Data:824633779184 Len:4 Cap:5}，&[10 0 0 1]
```

## 6.深度拷贝

Golang中切片的传递，底层数组是浅拷贝（操作原来切片会影响新切片），那么有没有深度拷贝的方法呢？

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	case1()
}

func case1() {
	var s1 = []int{1, 2, 3, 4}
	var s2 = make([]int, 5)
	copy(s2, s1)//按长度最短的copy
	printSliceStruct(&s1)
	printSliceStruct(&s2)
}

func printSliceStruct(s *[]int) {
	/*
		reflect.SliceHeader：用来描述切片的底层实现
		unsafe.Pointer：是一个通用类型的指针，可以用作不同类型指针相互转换
	*/
	ss := (*reflect.SliceHeader)(unsafe.Pointer(s))
	fmt.Printf("%+v，%v\n", ss, s)
}
```
输出结果：

```
&{Data:824633803200 Len:4 Cap:4}，&[1 2 3 4]
&{Data:824633779184 Len:5 Cap:5}，&[1 2 3 4 0]
```

## 7.面试题

```go
package main

import (
	"fmt"
)

func main() {
	doAppend := func(s []int) {
		s = append(s, 1)
		printSliceStruct(s)
	}
	s := make([]int, 8, 8)
	doAppend(s[:4])
	printSliceStruct(s)
	doAppend(s)
	printSliceStruct(s)
}

func printSliceStruct(s []int) {
	fmt.Println(s)
	fmt.Printf("len=%d cap=%d\n", len(s), cap(s))
}
```
输出结果：

```
[0 0 0 0 1]
len=5 cap=8
[0 0 0 0 1 0 0 0]
len=8 cap=8
[0 0 0 0 1 0 0 0 1]
len=9 cap=16
[0 0 0 0 1 0 0 0]
len=8 cap=8
```

