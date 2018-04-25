---
title: leetcode 46. Permutations 
date: 2018-04-25 21:14:15
tags: permutations
categories: leetcode
---
# Problem description

Given a collection of distinct integers, return all possible permutations.

# Example

Input: [1,2,3]
Output:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]

# 解题思路

复习一下全排列算法。
例如：
1,2,3
如果构造以1开头的全排列，那么可以基于`2,3`的全排列。`2,3`全排列又可以分为基于`2`和`3`开头的全排列。如果以`2`开头，那么剩下的`3`就是自己的全排列。如果以`3`开头，那么剩下的`2`就是自己的全排列。通过不断减小问题的规模，递归解决问题。

# 解题代码

``` go
func permute(nums []int) (result [][]int) {
	if len(nums) <= 1 {
		return [][]int{{nums[0]}}
	}
	for i := 0; i < len(nums); i++ {
		nums[0], nums[i] = nums[i], nums[0]
		t := permute(nums[1:])
		nums[0], nums[i] = nums[i], nums[0]
		for a := range t {
			result = append(result, append(t[a], nums[i]))
		}
	}
	return result
}
```
