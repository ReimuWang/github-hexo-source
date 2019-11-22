---
title: Leetcode Algorithms-28.Implement strStr()
date: 2018-09-14 16:53:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Implement strStr() - LeetCode](https://leetcode.com/problems/implement-strstr/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/28Implement strStr/0.jpg)

<!-- more -->

# 解法

显然，出题人希望我们能自行实现这个功能，而不是直接调用indexOf()或startsWith()。

**代码**

```
public class Solution {

    public int strStr(String haystack, String needle) {
        if (null == haystack) throw new NullPointerException("haystack is null");
        if (null == needle) return -1;
        int hl = haystack.length();
        int nl = needle.length();
        if (nl == 0) return 0;
        if (nl > hl) return -1;
        for (int hStart = 0; hStart <= hl - nl; hStart++) {
            String temp = haystack.substring(hStart, hStart + nl);
            if (needle.equals(temp)) return hStart;
        }
        return -1;
    }
}
```