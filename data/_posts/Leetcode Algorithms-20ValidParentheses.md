---
title: Leetcode Algorithms-20.Valid Parentheses
date: 2018-05-30 17:29:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Valid Parentheses - LeetCode](https://leetcode.com/problems/valid-parentheses/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/20ValidParentheses/0.jpg)

<!-- more -->

# 解法

**思路**

利用Java API提供的栈完成匹配

**代码**

```
import java.util.HashMap;
import java.util.Map;
import java.util.Stack;

public class Solution {

    private static final Map<String, String> MAP = new HashMap<String, String>();

    static {
        Solution.MAP.put(")", "(");
        Solution.MAP.put("}", "{");
        Solution.MAP.put("]", "[");
    }

    public boolean isValid(String s) {
        if (null == s)
            throw new NullPointerException("s is null");
        if ("".equals(s)) return true;
        Stack<String> stack = new Stack<String>();
        for (int i = 0; i < s.length(); i++) {
            String now = s.substring(i, i + 1);
            if (!Solution.MAP.containsKey(now))
                stack.push(now);
            else {
                if (stack.size() == 0) return false;
                String last = stack.pop();
                if (!Solution.MAP.get(now).equals(last)) return false;
            }
        }
        return stack.size() == 0;
    }
}
```