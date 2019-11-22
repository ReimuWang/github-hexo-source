---
title: Leetcode Algorithms-66.Plus One
date: 2018-09-17 16:24:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Plus One - LeetCode](https://leetcode.com/problems/plus-one/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/66Plus One/0.jpg)

<!-- more -->

# 解法

```
public class Solution {

    public int[] plusOne(int[] digits) {
        if (null == digits || digits.length == 0) return digits;
        int now = digits.length - 1;
        while (now >= 0) {
            if (digits[now] != 9) {
                digits[now]++;
                break;
            }
            digits[now--] = 0;
        }
        if (now == -1) {
            int[] result = new int[digits.length + 1];
            result[0] = 1;
            for (int i = 0; i < digits.length; i++) result[i + 1] = digits[i];
            return result;
        }
        return digits;
    }
}
```