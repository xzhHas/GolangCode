---
title: 二叉树用切片和链表实现以及写一个二叉树库
shortTitle: 23.用Go实现二叉树库
description: 二叉树用切片和链表实现，写一个二叉树库，数据结构库
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-04-22
---

## 前言

- 完全二叉树
  - 最底层节点按顺序从左到右排列。
- 满二叉树
  - 一颗二叉树只有0度和2度的节点。
- 二叉搜索树
  - 左子树上的所有节点的值均小于根节点的值。
  - 右子树上的所有节点的值均大于根节点的值。
- 平衡二叉搜索树
  - 左右两个子树的高度差的绝对值不超过1 。

二叉树的存储方式有链式存储和数组存储。（线索二叉树、红黑树等）

### 1、链表存储方式

```go
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func NewTreeNode(val int) *TreeNode {
	return &TreeNode{Val: val}
}
```

### 2、数组存储方式

```
	// 完全二叉树：         1
	//                  /   \
	//                 2     3
	//                / \   / \
	//               4   5 6   7
	// 以下为前中后序遍历,以下例子也是这个结果
	//	1245367 
	//	4251637
	//	4526731
```

左子树：2 * i + 1 

右子树：2 * i + 2

（i是数组的下标），元素值为arr[ 2 * i + 1 ]或arr[ 2 *  i + 2 ]

接下来将讲解二叉树的几种遍历方式，我全篇使用链式存储结构。

## 一、深度优先遍历

### 1、前序遍历

1、递归遍历

```go
// 前序遍历：根 -> 左 -> 右
func preorderTraversal(root *TreeNode) {
	if root != nil {
		fmt.Println(root.Val)         // 访问根节点
		preorderTraversal(root.Left)  // 递归遍历左子树
		preorderTraversal(root.Right) // 递归遍历右子树
	}
}
```

2、迭代遍历

深度优先遍历的递归版本都是简洁易读的，相较于迭代版本，更直观。迭代版本使用到了一种数据结构栈，以下我使用的栈是自己封装的库函数，如果有感兴趣的朋友，可以看shard库介绍，写shard库主要还是由于Golang没提供更多的数据结构模版。

```go
// 前序遍历：根 -> 左 -> 右（迭代实现）
func preorderTraversal(root *TreeNode) {
	if root == nil {
		return
	}
	// 栈存放的全是 *TreeNode
	s := shard.NewStackArray[*TreeNode]()
	s.Push(root)
	for s.Len() > 0 {
		// 栈顶弹出并删除
		node, _ := s.Pop()
		fmt.Println(node.Val)
		// 先压右子节点，再压左子节点，因为栈是后进先出（LIFO）
		if node.Right != nil {
			s.Push(node.Right)
		}
		if node.Left != nil {
			s.Push(node.Left)
		}
	}
}
```

### 2、中序遍历

1、递归遍历

```go
// 中序遍历：左 -> 根 -> 右
func inorderTraversal(root *TreeNode) {
	if root != nil {
		inorderTraversal(root.Left)  // 递归遍历左子树
		fmt.Println(root.Val)        // 访问根节点
		inorderTraversal(root.Right) // 递归遍历右子树
	}
}
```

2、迭代遍历

```go
// 中序遍历：左 -> 根 -> 右（迭代实现）
func inorderTraversal(root *TreeNode) {
	if root == nil {
		return
	}
	// 栈存放的全是 *TreeNode
	s := shard.NewStackArray[*TreeNode]()
	cur := root
	for cur != nil || !s.IsEmpty() {
		for cur != nil {
			s.Push(cur)
			cur = cur.Left
		}
		node, _ := s.Pop()
		fmt.Println(node.Val)
		cur = node.Right
	}
}
```

### 3、后序遍历

1、递归遍历

```go
// 后序遍历：左 -> 右 -> 根
func postorderTraversal(root *TreeNode) {
	if root != nil {
		postorderTraversal(root.Left)  // 递归遍历左子树
		postorderTraversal(root.Right) // 递归遍历右子树
		fmt.Println(root.Val)          // 访问根节点
	}
}
```

2、迭代遍历

```go
// 后序遍历：左 -> 右 -> 根（迭代实现）
func postorderTraversal(root *TreeNode) {
	if root == nil {
		return
	}
	// 栈存放的全是 *TreeNode
	s1 := shard.NewStackArray[*TreeNode]()
	s1.Push(root)
	s2 := shard.NewStackArray[*TreeNode]()
	for !s1.IsEmpty() {
		node, _ := s1.Pop()
		s2.Push(node)
		if node.Left != nil {
			s1.Push(node.Left)
		}
		if node.Right != nil {
			s1.Push(node.Right)
		}
	}
	for !s2.IsEmpty() {
		node, _ := s2.Pop()
		fmt.Println(node.Val)
	}
}
```

## 二、广度优先遍历

### 1、层序遍历

```go
// 层序遍历
func postorderTraversal(root *TreeNode) {
	if root == nil {
		return
	}
	q := shard.NewQueueArray[*TreeNode]()
	q.Enqueue(root)
	for !q.IsEmpty() {
		node, _ := q.Dequeue()
		fmt.Print(node.Val, " ")
		if node.Left != nil {
			q.Enqueue(node.Left)
		}
		if node.Right != nil {
			q.Enqueue(node.Right)
		}
	}
}
```

## 三、shard库介绍

GitHub链接：[https://github.com/xzhHas/shard](https://github.com/xzhHas/shard)

shard库获取：

```
go get -u github.com/xzhHas/shard@latest
```

关于使用Golang写一个数据结构的库，目前只支持栈、队列、堆。


![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181834418.png)

