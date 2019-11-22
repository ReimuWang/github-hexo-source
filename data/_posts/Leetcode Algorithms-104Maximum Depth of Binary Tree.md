---
title: Leetcode Algorithms-104.Maximum Depth of Binary Tree
date: 2018-09-19 10:46:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Maximum Depth of Binary Tree - LeetCode](https://leetcode.com/problems/maximum-depth-of-binary-tree/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/104Maximum Depth of Binary Tree/0.jpg)

<!-- more -->

# 解法

```
public class Solution {

    public int maxDepth(TreeNode root) {
        return null == root ? 0 : Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
    }
}

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}
```