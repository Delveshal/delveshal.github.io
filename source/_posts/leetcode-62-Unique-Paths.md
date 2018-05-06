---
title: leetcode 62. Unique Paths
date: 2018-05-06 20:21:55
tags: dp
categories: leetcode
---

# Description

A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).

The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).

How many possible unique paths are there?

![image](https://leetcode.com/static/images/problemset/robot_maze.png)
Above is a 7 x 3 grid. How many possible unique paths are there?

Note: m and n will be at most 100.

<!-- more -->

# Example1

```
Input: m = 3, n = 2
Output: 3
Explanation:
From the top-left corner, there are a total of 3 ways to reach the bottom-right corner:
1. Right -> Right -> Down
2. Right -> Down -> Right
3. Down -> Right -> Right
```
# Example2

```
Input: m = 7, n = 3
Output: 28
```

# 题解
```
var ans [101][101]int
func uniquePaths(m int, n int) int {
    if ans[m][n] != 0{
        return ans[m][n]
    }
    if m ==1 || n == 1 {
        return 1
    }
    ans[m][n] = uniquePaths(m-1,n) + uniquePaths(m,n-1)
    return ans[m][n]
}
```
