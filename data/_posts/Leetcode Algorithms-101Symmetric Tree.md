---
title: Leetcode Algorithms-101.Symmetric Tree
date: 2018-09-18 17:21:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Symmetric Tree - LeetCode](https://leetcode.com/problems/symmetric-tree/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/101Symmetric Tree/0.jpg)

<!-- more -->

# 解法

```
public class Solution {

    public boolean isSymmetric(TreeNode root) {
        return root == null || isSymmetricHelp(root.left, root.right);
    }

    private boolean isSymmetricHelp(TreeNode left, TreeNode right) {
        if(left == null || right == null) return left == right;
        if(left.val != right.val) return false;
        return isSymmetricHelp(left.left, right.right) && isSymmetricHelp(left.right, right.left);
    }
}

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}
```