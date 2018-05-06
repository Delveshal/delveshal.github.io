---
title: leetcode 79. Word Search
date: 2018-05-06 17:37:16
tags: [dfs]
---

# Description
Given a 2D board and a word, find if the word exists in the grid.

The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.

# Example
```
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false.
```

# 解题思路
深搜，标记走过的路径使用特殊字符

<!-- more -->
# 题解

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
