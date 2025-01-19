---
title: defer和return竟然区别这么大？
shortTitle: 3.defer和return
description: 多个defer的顺序？defer闭包问题？
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-02-21
star: true
order: -8
---
## 1.多个defer的顺序？
后进先出。


## 2.defer与return

```go
package main

import "fmt"

func main() {
	var n int
	n = 1
	defer fmt.Println(n)
	defer func() {
		fmt.Println(n)
	}()
	defer func(n int) {
		fmt.Println(n)
	}(n)
	n = 2
	return
}
```
输出结果：

```
1
2
1
```
为什么？

defer语句在声明时，会立即捕获当前的变量值，并将这些值保存，然后等外围函数返回之后，在后进先出执行defer语句。又因为第三个和第一个都是传值的，所以固定n就是1 。但是第二个由于是匿名函数，直接是引用了 n，而不是在defer声明是捕获n，因此在执行这个defer时，会读取当前引用的n的值。

```go
package main

import "fmt"

func main() {
	a := [4]int{1, 2, 3, 4}
	defer pring(&a)
	a[0] = 1234
	return
}
func pring(a *[4]int) {
	for i := range a {
		fmt.Println(a[i])
	}
}
```
输出结果：

```
1234
2
3
4
```
why？

我们defer时传递的是数组的地址，地址不变，但是地址上的内容被修改了，所以输出会被修改。

```go
package main

import "fmt"

func main() {
	res := df()
	fmt.Println(res)
}

func df() (res int) {
	n := 1
	defer func() {
		res++
	}()
	return n
}
```
输出结果：

```
2
```
为什么？

defer与return的操作时机是：
1.设置返回值
2.执行defer语句
3.将结果返回

所以，显示设置返回值res=1，然后执行defer，最后返回res=2 。

```go
package main

import "fmt"

func main() {
	res := df()
	fmt.Println(res)
}

func df() int {
	n := 1
	defer func() {
		n++
	}()
	return n
}
```
如果返回值，未定义，那么返回的是n=1的值。

**注意：**

 1. defer定义的延迟函数的参数在声明defer语句当时就已经确定下来了
 2. return不是原子级操作，执行过程是：设置返回值->执行defer语句->奖结果返回


## 3.异常捕获

 1. recover()只能恢复当前函数级或以当前函数为首的调用链中的panic()，恢复后调用当前函数结束，但是调用此函数的函数继续执行
 2. 函数发生了panic之后会一直向上传递，如果直至main函数都没有recover()，程序将终止，如果碰见了recover()，将被捕获。

```go
package main

import "fmt"

func main() {
	d1()
}

func d1() {
	fmt.Println("d1上")
	d2()
	fmt.Println("d1下")
}

func d2() {
	defer func() {
		recover()
	}()
	fmt.Println("d2上")
	d3()
	fmt.Println("d2下")
}
func d3() {
	fmt.Println("d3上")
	panic("err")
	fmt.Println("d3下")
}
```

输出结果：

```
d1上
d2上
d3上
d1下
```

