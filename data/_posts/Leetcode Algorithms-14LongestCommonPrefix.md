---
title: Leetcode Algorithms-14.Longest Common Prefix
date: 2018-05-24 17:32:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Longest Common Prefix - LeetCode](https://leetcode.com/problems/longest-common-prefix/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/14LongestCommonPrefix/0.jpg)

<!-- more -->

# 解法

**思路**

首先，找到strs[0]与strs[1]之间的最长公共前缀，不妨命名为x。

而后再找到x与strs[2]之间的最长公共前缀，将x赋值为新找到的前缀。

...

依此类推，直至结束。

过程中任何时候x为""则停止循环，因为此时已不可能再找到更短的最长公共前缀了。

**代码**

```
public class Solution {

    public String longestCommonPrefix(String[] strs) {
        if (null == strs || strs.length == 0) return "";
        if (strs.length == 1) return strs[0];
        String model = strs[0];
        for (int i = 1; i < strs.length; i++) {
            String now = strs[i];
            while (!now.startsWith(model))
                model = model.substring(0, model.length() - 1);
            if ("".equals(model)) break;
        }
        return model;
    }
}
```