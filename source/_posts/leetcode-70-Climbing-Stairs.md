---
title: leetcode 70. Climbing Stairs
date: 2018-05-06 20:14:05
tags:
categories: leetcode
---

# Description

You are climbing a stair case. It takes n steps to reach to the top.

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

Note: Given n will be a positive integer.

<!-- more -->

# Example1
```
Input: 2
Output: 2
Explanation: There are two ways to climb to the top.
1. 1 step + 1 step
2. 2 steps
```
# Example 2
```
Input: 3
Output: 3
Explanation: There are three ways to climb to the top.
1. 1 step + 1 step + 1 step
2. 1 step + 2 steps
3. 2 steps + 1 step
```

# 题解
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
