---
title: 学习B-树
date: 2018-04-22 21:29:21
tags: [tree,balance-tree]
categories: [tree]
mathjax: true
---

# 学习B-树

B-树（balance-tree）

# B-树的定义
一颗 m 阶的 B-树， 或为空树， 或为满足下列特性的 m 叉树：
1. 树中每个节点至多有 m 颗子树
2. 若根节点不是叶子节点，则至少有两颗子树
3. 除根之外的所有非终端节点至少有⌈m/2⌉颗子树，则至少有$\lceil m/2\rceil-1$个关键字（$\lceil m/2\rceil$ 为向上取整）
4. 所有子节点都出现在同一层次上，并且不带消息，通常称为失败结点。
5. 所有非终端节点最多有 m-1 个关键字

关键字个数n必须满足：$\lceil m/2\rceil-1 \le n \le m-1$

![balance-tree](https://upload-images.jianshu.io/upload_images/8053527-2d1640a9a29b6856.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 一棵含有N个总关键字数的m阶的B树的最大高度是多少?
设 m 阶B-树的高度为h+1。（+1表示最后一层叶子结点）
根据B-树的定义， 第一层至少有一个结点；第二层有2个结点，第三层有$2\times\lceil m/2\rceil$个结点，第四层有$2\times\lceil m/2\rceil^2$个结点，以此类推，第 h+1 层有$2\times\lceil m/2\rceil ^{h-1}$个结点
若 m 阶 B-树中具有 N 个关键字，则叶子结点既查找不成功的结点为 N+1.（如上图，失败结点有$3\times7+2=23$个，注意两个灰色结点）

$$N+1\ge 2 \times \lceil m/2\rceil ^{h-1}$$
$$h \le log_{\lceil m/2\rceil}(\frac{N+1}{2})+1$$
