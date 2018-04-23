---
title: leetcode 31. Next Permutation
date: 2018-04-05 19:37:28
tags: [leetcode,permutation]
categories: leetcode
---

# leetcode 31. Next Permutation

Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.

If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).

The replacement must be in-place, do not allocate extra memory.

Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1

# 解题思路
<!-- more -->

题目的意思的找到当前全排列的下一个排列序列。
例如 1,2,3 的全排列
1,2,3
1,3,2
2,1,3
2,3,1
3,1,2
3,2,1

那么 1,2,3 下一个就是 1,3,2。 
**规定**： 最后一个的下一个是第一个。 3,2,1 -> 1,2,3

首先，很明显可以发现：如果一个序列是非递增，那么这个序列是不能再重排，获得一个更大的数。
如果把非递增数列看成序列的子结构，那么这一部分将不可以通过重排获得更大的数，只能加入一位数参与重排。重排的方法是从后往前找到第一个前面一个小于后面一个，即a[i] > a[i-1]，a[k] <= a[k-1],k > i。 那么把a[i] 与右边的序列重排，如果要使得序列变大同时又尽可能小，那么只能选比a[i]刚好大的a[j]，两个交换，这时候还不是最小的，所以要把非递增序列从小到大排列，即把非递增序列反转.
例如：
`1,2,4,3` ，那么 `4,3` 是一个部分非递增序列，只能加入`2`参与重排。`3` 刚好比 `2` 大，两个交换变成`1,3,4,2`，然后反转成`1,3,2,4`.

go的实现：
``` go
func nextPermutation(nums []int)  {
    i := len(nums) - 2
    for i>=0 && nums[i] >= nums[i+1]{
        i--
    }
    
    if i>=0 {
        j := len(nums) - 1
        for j>=0 && nums[j] <= nums[i]{
            j--
        }
        nums[j],nums[i] = nums[i],nums[j]
    }
    nums = nums[i+1:]
    for k := 0;k<len(nums)/2;k++{
        nums[k],nums[len(nums) - 1 - k] = nums[len(nums) - 1 -k],nums[k]
    }
}
```

中间一步可以使用二分查找。不过在leetcode上都是4ms。

``` go
func nextPermutation(nums []int) {
	i := len(nums) - 2
	for i>=0 && nums[i] >= nums[i+1]{
		i--
	}

	if i>=0 {
		j := binary(nums[i:],nums[i]) + i
		nums[j],nums[i] = nums[i],nums[j]
	}
	nums = nums[i+1:]
	for k := 0;k<len(nums)/2;k++{
		nums[k],nums[len(nums) - 1 - k] = nums[len(nums) - 1 -k],nums[k]
	}
}

func binary(nums []int,target int) int {
	left := 0
	right := len(nums)
	for left < right - 1 {
		mid := (right + left) / 2
		if nums[mid] > target {
			left = mid
		}else{
			right = mid
		}
	}
	return left
}
```
