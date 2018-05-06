---
title: leetcode 75. Sort Colors
date: 2018-05-06 20:09:20
tags: [sort]
categories: [leetcode]
---

# Description
Given an array with n objects colored red, white or blue, sort them in-place so that objects of the same color are adjacent, with the colors in the order red, white and blue.

Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.

Note: You are not suppose to use the library's sort function for this problem.

<!-- more -->

# Example
Input: [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]
# Follow up
* A rather straight forward solution is a two-pass algorithm using counting sort.
First, iterate the array counting number of 0's, 1's, and 2's, then overwrite array with total number of 0's, then 1's and followed by 2's.

* Could you come up with a one-pass algorithm using only constant space?

# 题解

```
func sortColors(nums []int)  {
	j := len(nums) - 1
	for i := 0;i<len(nums) && i < j;i++{
		if nums[i] >= 2{
			for j>=0 && nums[j] == 2 {
				j--
			}
			if i < j && j>=0 {
				nums[i],nums[j]=nums[j],nums[i]
			}
		}
	}
	for i := 0;i<j;i++{
		if nums[i] >= 1{
			for j>=0 && nums[j] >= 1 {
				j--
			}
			if i < j && j>=0{
				nums[i],nums[j]=nums[j],nums[i]
			}
		}
	}
}
```

```
func sortColors(nums []int) {
	p1 := 0
	p2 := len(nums) - 1
	for i := 0; i <= p2; {
		switch nums[i] {
		case 0:
			nums[i], nums[p1] = nums[p1], nums[i]
			p1++
			i++
		case 1:
			i++
		case 2:
			nums[i], nums[p2] = nums[p2], nums[i]
			p2--
		}
	}
}
```
