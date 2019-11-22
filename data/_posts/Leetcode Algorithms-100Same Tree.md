---
title: Leetcode Algorithms-100.Same Tree
date: 2018-09-18 15:10:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Same Tree - LeetCode](https://leetcode.com/problems/same-tree/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/100Same Tree/0.jpg)

<!-- more -->

# 解法

```
public class Solution {

    public boolean isSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) return true;
        if (p == null || q == null) return false;
        if (p.val != q.val) return false;
        return this.isSameTree(p.left, q.left) && this.isSameTree(p.right, q.right);
    }
}

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}
```