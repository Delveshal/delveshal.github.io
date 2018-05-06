---
title: leetcode 78. Subsets
date: 2018-05-06 17:42:03
tags: [set]
categories: [leetcode]
---

# Description

Given a set of distinct integers, nums, return all possible subsets (the power set).

Note: The solution set must not contain duplicate subsets.

# Example

```
Input: nums = [1,2,3]
Output:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```
# 解题思路

递归，缩小问题规模思想。例如[1,2,3]集合的子集可以**由[1,2]集合的子集**和**[1,2]集合的子集加上[3]**构成。
<!-- more -->
```
[1,2]:
[
  [],
  [1],
  [2],
  [1,2]
]
加上[3]
[1,2,3]:
[
  []
  [1],
  [2],
  [1,2],    //这前面与[1,2]子集一致，这后面使用前面的加上[3]
  [3],
  [1,3],
  [2,3],
  [1,2,3]
]
```

# 题解

```
func subsets(nums []int)(result [][]int){
	if len(nums) == 0{
		result = append(result,[]int{})
		return
	}
	result = subsets(nums[:len(nums) - 1])
	t := make([][]int,len(result))
	for i := range result{
		t[i] = append(t[i],nums[len(nums)-1])
		t[i] = append(t[i],result[i]...)
	}
	result = append(result,t...)
	return
}
```
