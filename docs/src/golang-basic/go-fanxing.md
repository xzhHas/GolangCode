---
title: 泛型编程
shortTitle: 15.泛型编程
description: 泛型编程，是一种允许你编写出来可以处理不同类型数据的代码的程序设计范式。有助于提高代码复用性，增加类型安全性，有时还可以优化性能。
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-04-29
---


## 一、简单介绍一下Go的泛型

泛型编程，是一种允许你编写出来可以处理不同类型数据的代码的程序设计范式。有助于提高代码复用性，增加类型安全性，有时还可以优化性能。

Golang中泛型：

```go
type List[T any] struct{}
```

T：类型参数，any：类型约束，T可以是任意类型。

## 二、为什么需要泛型？

1、类型安全

原本Golang是没有泛型的，通常是使用interface{}，但是这样做会失去类型检查的好处。使用泛型强类型可以在编译期进行类型检查，但是interface{}弱类型不可以。

2、代码复用

3、性能优化

Go编译器会在编译期间为每个泛型参数生成具体的实现，因此，运行时不需要进行额外的类型检查或转换，有助于优化性能。

## 三、Go泛型基础

类型参数

### 基础语法

在Go中，泛型的类型参数通常使用方括号进行声明，紧随函数或结构体名称之后。

```go
func Add[T any](a, b T) T {
    return a + b
}
```

这里，T 是一个类型参数，并且使用了 any 约束，意味着它可以是任何类型。

多类型参数

Go泛型不仅支持单一的类型参数，你还可以定义多个类型参数。

```cpp
func Pair[T, U any](a T, b U) (T, U) {
    return a, b
}
```

在这个例子中，Pair 函数接受两个不同类型的参数 a 和 b，并返回这两个参数。

类型约束

内建约束

Go内置了几种类型约束，如 any，表示任何类型都可以作为参数。

```cpp
func PrintSlice[T any](s []T) {
    for _, v := range s {
        fmt.Println(v)
    }
}
```

自定义约束

除了内置约束，Go还允许你定义自己的约束。这通常是通过接口来实现的。

```cpp
type Addable interface {
    int | float64
}
​
func Add[T Addable](a, b T) T {
    return a + b
}
```

这里，Addable 是一个自定义的类型约束，只允许 int 或 float64 类型。

泛型函数与泛型结构体

### 泛型函数

我们已经看到了几个泛型函数的例子，它们允许你在多种类型上执行相同的逻辑。

```cpp
func Max[T comparable](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

### 泛型结构体

除了函数，Go也支持泛型结构体。

```cpp
type Box[T any] struct {
    Content T
}
```

这里，Box 是一个泛型结构体，它有一个 Content 字段，类型为 T。

泛型方法

在泛型结构体中，你还可以定义泛型方法。

```cpp
func (b Box[T]) Empty() bool {
    return b.Content == nil
}
```

举例：

```cpp
type Person[T any] struct {
    Name string
    data T
}
​
func (p *Person[T]) Printing() {
    fmt.Println(p.Name, p.data)
}
​
func (p *Person[T]) Set(name string, data T) {
    p.Name = name
    p.data = data
}
​
type Base struct {
    ID   int
    Name string
}
​
func main() {
    p := Person[int]{}
    q := Person[string]{}
    d := Person[Base]{}
    p.Set("Tom", 1)
    p.Printing()
    q.Set("Jerry", "hello")
    q.Printing()
    d.Set("Jack", Base{ID: 1, Name: "Mar"})
    d.Printing()
}
```


// 输出结果

```cpp
Tom 1
Jerry hello
Jack {1 Mar}
```

## 四、Go泛型高级

后续学习了在更新。。。

## 五、最后

如果以上内容对你有帮助的话也可以关注我的公众号“GolangCode”，获取更多技术干货。

![GolangCode](https://cdn.golangcode.cn/images/202501171944968.png)
