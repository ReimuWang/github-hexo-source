---
title: Leetcode Algorithms-38.Count and Say
date: 2018-09-14 18:52:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Count and Say - LeetCode](https://leetcode.com/problems/count-and-say/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/38Count and Say/0.jpg)

<!-- more -->

# 解法

**代码**

```
public class Solution {

    public String countAndSay(int n) {
        if (n < 1 || n > 30) return null;
        String str = "1";
        for (int i = 1; i < n; i++) {
            StringBuilder sb = new StringBuilder();
            int mark = 0;
            while (mark < str.length()) {
                int count = numberCount(str, mark);
                sb.append(count).append(str.charAt(mark));
                mark += count;
            }
            str = sb.toString();
        }
        return str;
    }

    private int numberCount(String str, int mark) {
        int count = 1;
        while (mark < str.length() - 1) {
            if (str.charAt(mark + 1) != str.charAt(mark)) break;
            count++;
            mark++;
        }
        return count;
    }
}
```