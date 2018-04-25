---
title: leetcode 48. Rotate Image
date: 2018-04-25 21:22:07
tags: [in-place]
categories: leetcode
---
# Problem description

You are given an n x n 2D matrix representing an image.

Rotate the image by 90 degrees (clockwise).

**Note:**

You have to rotate the image [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm), which means you have to modify the input 2D matrix directly. **DO NOT** allocate another 2D matrix and do the rotation.

# Example 1:
```
Given input matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],

rotate the input matrix in-place such that it becomes:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```
# Example 2:
```
Given input matrix =
[
  [ 5, 1, 9,11],
  [ 2, 4, 8,10],
  [13, 3, 6, 7],
  [15,14,12,16]
], 

rotate the input matrix in-place such that it becomes:
[
  [15,13, 2, 5],
  [14, 3, 4, 1],
  [12, 6, 8, 9],
  [16, 7,10,11]
]
```
# 解题思路

题目说了顺时针旋转，那么就把矩阵一圈一圈地顺时针转，然后推出以下公式：
```
        matrix[x][y],matrix[y][n-x-1] = matrix[y][n-x-1],matrix[x][y]
        matrix[x][y],matrix[n-x-1][n-y-1] = matrix[n-x-1][n-y-1],matrix[x][y]
        matrix[x][y],matrix[n-y-1][x] = matrix[n-y-1][x],matrix[x][y]
```

# 解题代码

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
