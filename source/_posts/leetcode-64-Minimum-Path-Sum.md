---
title: leetcode 64. Minimum Path Sum
date: 2018-05-06 20:17:38
tags: dp
categories: leetcode
---

# Description
Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.

Note: You can only move either down or right at any point in time.

# Example

```
Input:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
Output: 7
Explanation: Because the path 1→3→1→1→1 minimizes the sum.
```

# 解题思路
某个格子只能从它的上方和左方来，所以只要最小的那个一步步构造就能得出最优解。
<!-- more -->
# 题解
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
