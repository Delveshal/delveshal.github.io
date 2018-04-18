---
title: leetcode 33. Search in Rotated Sorted Array
date: 2018-04-06 14:25:22
tags: [leetcode,binary search]
categories: leetcode
---

# leetcode 33. Search in Rotated Sorted Array

Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.

(i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).

You are given a target value to search. If found in the array return its index, otherwise return -1.

You may assume no duplicate exists in the array.

# 解题思路
* 第一步，通过二分查找找到翻转点。一开始我是通过比较a[mid]>a[mid+1]，如果是则是翻转点，否则检查左边是否a[left]>a[mid]和右边是否a[mid]>a[right]，如果左边符合递归检查左边，否则检查右边。
例如： `2,4,5,6,7,0,1` 明显`6`不符合。检查左边`2,4,5`，不符合。检查右边`7,0,1`，符合。`0`不符合，`7` 大于 `0` 是翻转点。

* 第二步，把两段排好序的序列进行二分查找。

``` go
func search(nums []int, target int) int {
	pivot := find(nums)
	if pivot == -1 {
		return binary(nums,target)
	}
	var ans int
	if ans = binary(nums[:pivot + 1],target);ans != -1{
		return ans
	}
	if ans = binary(nums[pivot + 1:],target);ans == -1{
		return ans
	}
	ans = pivot + 1 + ans
	return ans
}

func find(nums []int) int {
	if len(nums) == 0 {
		return -1
	}
	mid := (len(nums)-1)/2
	if mid + 1 >= len(nums){
		return -1
	}
	if nums[mid] > nums[mid+1]{
		return mid
	}
	if nums[0] > nums[mid]{
		return find(nums[:mid + 1])
	}
	if nums[mid] > nums[len(nums)-1]{
		return mid + find(nums[mid:])
	}
	return -1
}

func binary(nums []int,target int) int {
	left := 0
	right := len(nums) - 1
	for left <= right {
		mid := (right + left) / 2
		if nums[mid] == target{
			return mid
		}
		if nums[mid] < target {
			left = mid + 1
		}else{
			right = mid -1
		}
	}
	return -1
}
```

参考了别人的寻找翻转点的方法，发现别人的思路实现起来更精简优雅。

``` go
func search(nums []int, target int) int {
	pivot := find(nums)
	var ans int
	if ans = binary(nums[:pivot],target);ans != -1{
		return ans
	}
	if ans = binary(nums[pivot:],target);ans == -1{
		return ans
	}
	return pivot + ans
}

func find(nums []int) int {
	left := 0
	right := len(nums) - 1
	for left < right {
		mid := (left + right) / 2
		if nums[mid] > nums[right]{
			left = mid + 1
		}else{
			right = mid
		}
	}
	return left
}

func binary(nums []int,target int) int {
	left := 0
	right := len(nums) - 1
	for left <= right {
		mid := (right + left) / 2
		if nums[mid] == target{
			return mid
		}
		if nums[mid] < target {
			left = mid + 1
		}else{
			right = mid -1
		}
	}
	return -1
}
```

利用sort包的二分查找版本

``` go
func search(nums []int, target int) int {
	pivot := find(nums)
	var ans int
	if ans = sort.SearchInts(nums[:pivot], target); ans < len(nums) && nums[ans] == target {
		return ans
	}
	if ans = sort.SearchInts(nums[pivot:], target); ans+pivot >= len(nums) || nums[pivot+ans] != target {
		return -1
	}
	return pivot + ans
}

func find(nums []int) int {
	left := 0
	right := len(nums) - 1
	for left < right {
		mid := (left + right) / 2
		if nums[mid] > nums[right] {
			left = mid + 1
		} else {
			right = mid
		}
	}
	return left
}
```
