---
title: leetcode 题解（一）
date: 2018-05-14 22:45:27
tags:
categories: leetcode
---
# leetcode 31. Next Permutation

## 解题思路

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
<!-- more -->
## 解题代码
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
# leetcode 33. Search in Rotated Sorted Array

## 解题思路
* 第一步，通过二分查找找到翻转点。一开始我是通过比较a[mid]>a[mid+1]，如果是则是翻转点，否则检查左边是否a[left]>a[mid]和右边是否a[mid]>a[right]，如果左边符合递归检查左边，否则检查右边。
例如： `2,4,5,6,7,0,1` 明显`6`不符合。检查左边`2,4,5`，不符合。检查右边`7,0,1`，符合。`0`不符合，`7` 大于 `0` 是翻转点。

* 第二步，把两段排好序的序列进行二分查找。

## 解题代码
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
# leetcode 46. Permutations

## 解题思路

复习一下全排列算法。
例如：
1,2,3
如果构造以1开头的全排列，那么可以基于`2,3`的全排列。`2,3`全排列又可以分为基于`2`和`3`开头的全排列。如果以`2`开头，那么剩下的`3`就是自己的全排列。如果以`3`开头，那么剩下的`2`就是自己的全排列。通过不断减小问题的规模，递归解决问题。

## 解题代码

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

# leetcode 48. Rotate Image

## 解题思路

题目说了顺时针旋转，那么就把矩阵一圈一圈地顺时针转，然后推出以下公式：
```
        matrix[x][y],matrix[y][n-x-1] = matrix[y][n-x-1],matrix[x][y]
        matrix[x][y],matrix[n-x-1][n-y-1] = matrix[n-x-1][n-y-1],matrix[x][y]
        matrix[x][y],matrix[n-y-1][x] = matrix[n-y-1][x],matrix[x][y]
```

## 解题代码

``` go
func rotate(matrix [][]int)  {
    n := len(matrix)
    for y:=0;y<n-1;y++{
        for x := y;x<n-y-1;x++{
        matrix[x][y],matrix[y][n-x-1] = matrix[y][n-x-1],matrix[x][y]
        matrix[x][y],matrix[n-x-1][n-y-1] = matrix[n-x-1][n-y-1],matrix[x][y]
        matrix[x][y],matrix[n-y-1][x] = matrix[n-y-1][x],matrix[x][y]
        }
    }
}
```
# leetcode 56. Merge Intervals
## 解题代码
```
/**
 * Definition for an interval.
 * type Interval struct {
 *	   Start int
 *	   End   int
 * }
 */
func merge(intervals []Interval) []Interval {
	if len(intervals) <= 0{
		return intervals
	}
	sort.Slice(intervals, func(i, j int) bool {
		return intervals[i].Start < intervals[j].Start
	})
	result := make([]Interval,0,len(intervals))
	result = append(result,intervals[0])
	tail := len(result)-1
	for i:=1;i<len(intervals);i++{
        tail = len(result)-1
		if result[tail].End >= intervals[i].Start{
			if result[tail].End < intervals[i].End{
				result[tail].End = intervals[i].End
			}
		}else{
				result = append(result, intervals[i])
        }
	}
    return result
}
```
# leetcode 64. Minimum Path Sum

## 解题思路
某个格子只能从它的上方和左方来，所以只要最小的那个一步步构造就能得出最优解。
## 解题代码
```
func minPathSum(grid [][]int) int {
	if len(grid) <= 0{
		return 0
	}
	dp := make([][]int,len(grid))
	for i := range dp {
		dp[i] = make([]int,len(grid[0]))
	}
	dp[0][0] = grid[0][0]
	for i := 1;i<len(grid);i++{
		dp[i][0] = dp[i-1][0] + grid[i][0]
	}
	for j:=1;j<len(grid[0]);j++{
		dp[0][j] = dp[0][j-1] + grid[0][j]
	}
	for i := 1;i<len(grid);i++{
		for j:=1;j<len(grid[i]);j++{
			dp[i][j] = min(dp[i-1][j],dp[i][j-1]) + grid[i][j]
		}
	}
	return dp[len(grid)-1][len(grid[0])-1]
}

func min (a,b int) int{
	if a < b{
		return a
	}
	return b
}
```

# leetcode 70. Climbing Stairs
## 解题代码
```
var t [101]int
func climbStairs(n int) int {
    t[0] = 1
    t[1] = 1
    t[2] = 2
    if t[n] != 0{
        return t[n]
    }
    t[n-1] = climbStairs(n-1)
    t[n-2] = climbStairs(n-2)
    return t[n-1] + t[n-2]
}
```

# leetcode 75. Sort Colors
## 解题代码

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

# leetcode 78. Subsets

## 解题思路

递归，缩小问题规模思想。例如[1,2,3]集合的子集可以**由[1,2]集合的子集**和**[1,2]集合的子集加上[3]**构成。

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

## 解题代码

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

# leetcode 79. Word Search

## 解题思路
深搜，标记走过的路径使用特殊字符

## 解题代码

```
var dir = [4][]int{{0, 1}, {1, 0}, {-1, 0}, {0, -1}}

func exist(board [][]byte, word string) bool {
	for i := range board {
		for j := range board[i] {
			if board[i][j] != word[0] {
				continue
			}
			if dfs(board, word, i, j) {
				return true
			}
		}
	}
	return false
}

func dfs(board [][]byte, word string , startx, starty int) bool {
	var zeroByte byte
	if len(word) <= 0 || board[startx][starty] != word[0] {
		return false
	}
	if len(word) == 1 && board[startx][starty] == word[0] {
		return true
	}
	tmp := board[startx][starty]
	board[startx][starty] = zeroByte
	for i := range dir {
		tx := startx+dir[i][0]
		ty := starty+dir[i][1]
		if tx < 0 || tx >= len(board) {
			continue
		}
		if ty < 0 || ty >= len(board[0]) {
			continue
		}
		if dfs(board, word[1:],tx ,ty ){
			return true
		}
	}
	board[startx][starty] = tmp
	return false
}
```
