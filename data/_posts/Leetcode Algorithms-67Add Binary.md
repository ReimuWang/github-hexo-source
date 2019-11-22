---
title: Leetcode Algorithms-67.Add Binary
date: 2018-09-17 16:59:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Add Binary - LeetCode](https://leetcode.com/problems/add-binary/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/67Add Binary/0.jpg)

<!-- more -->

# 解法

```
public class Solution {

    public String addBinary(String a, String b) {
        if (null == a || a.length() == 0) return b;
        if (null == b || b.length() == 0) return a;
        StringBuilder sb = new StringBuilder();
        int ia = a.length() - 1;
        int ib = b.length() - 1;
        boolean mark = false;
        while (ia >= 0 || ib >= 0) {
            char ta = ia >= 0 ? a.charAt(ia) : '0';
            char tb = ib >= 0 ? b.charAt(ib) : '0';
            if (ta == '0' && tb == '0') {
                if (mark) sb.append('1');
                else sb.append('0');
                mark = false;
            } else if (ta == '1' && tb == '1') {
                if (mark) sb.append('1');
                else sb.append('0');
                mark = true;
            } else {
                if (mark) sb.append('0');
                else sb.append('1');
            }
            ia--;
            ib--;
        }
        if (mark) sb.append('1');
        return sb.reverse().toString();
    }
}
```