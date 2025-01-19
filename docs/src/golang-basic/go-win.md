---
title: Golang滑动窗口实现
shortTitle: 21.用Go实现滑动窗口算法
description: Golang滑动窗口实现
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-05-14
---

## 力扣：209. 长度最小的子数组

给定一个含有 n 个正整数的数组和一个正整数 target 。

找出该数组中满足其总和大于等于 target 的长度最小的 子数组 [numsl, numsl+1, ..., numsr-1, numsr] ，并返回其长度。如果不存在符合条件的子数组，返回 0 。

 

示例 1：
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。

示例 2：
输入：target = 4, nums = [1,4,4]
输出：1

示例 3：
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0

提示：
1 <= target <= 109
1 <= nums.length <= 105
1 <= nums[i] <= 105

进阶：
如果你已经实现 O(n) 时间复杂度的解法, 请尝试设计一个 O(n log(n)) 时间复杂度的解法。

---

**分析：** 根据上题可以直接想到使用双层循环就可以枚举所有的子数组来判断得到最小的子数组，但是这样的话时间复杂度为o(n2)，一般情况下都会超时。

所以，遇到类似题型，应该使用滑动窗口，就是双指针的策略。

因为题意是得到连续的子数组，所以我们可以设置一个快指针和一个慢指针，快指针不断向后移动，当满足sum>=target的时候，记录此时子数组的长度，让后移动slow指针，继续遍历子数组，不断获取最小长度minLen，最后返回minLen。


```go
// 209.长度最小的子数组
func minSubArrayLen(target int, nums []int) int {
	minLen := math.MaxInt
	i, j, sum := 0, 0, 0
	for j < len(nums) {
		sum += nums[j]
		for sum >= target {
			minLen = min(minLen, j-i+1)
			sum -= nums[i]
			i++
		}
		j++
	}
	if minLen < math.MaxInt {
		return minLen
	}
	return 0
}
```

