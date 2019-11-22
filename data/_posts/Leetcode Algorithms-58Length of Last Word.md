---
title: Leetcode Algorithms-58.Length of Last Word
date: 2018-09-17 15:59:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Length of Last Word - LeetCode](https://leetcode.com/problems/length-of-last-word/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/58Length of Last Word/0.jpg)

<!-- more -->

# 解法

```
public class Solution {

    public int lengthOfLastWord(String s) {
        if (null == s || s.length() == 0) return 0;
        String[] a = s.split(" ");
        return a.length == 0 ? 0 : a[a.length - 1].length();
    }
}
```