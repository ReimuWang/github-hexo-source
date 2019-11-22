---
title: Leetcode Algorithms-13.Roman to Integer
date: 2018-05-24 17:10:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Roman to Integer - LeetCode](https://leetcode.com/problems/roman-to-integer/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/13RomantoInteger/0.jpg)

![1.jpg](/images/blog_pic/Leetcode Algorithms/13RomantoInteger/1.jpg)

<!-- more -->

# 解法

**思路**

此题考校的是将一个不太熟悉的(一般大家都不会深究罗马数字的计算规则)概念抽象为代码的能力，正确理解题意即可。

**代码**

```
import java.util.HashMap;
import java.util.Map;

public class Solution {

    private static Map<Character, Integer> ROMAN_TO_ARAB = new HashMap<Character, Integer>();

    static {
        Solution.ROMAN_TO_ARAB.put('M', 1000);
        Solution.ROMAN_TO_ARAB.put('D', 500);
        Solution.ROMAN_TO_ARAB.put('C', 100);
        Solution.ROMAN_TO_ARAB.put('L', 50);
        Solution.ROMAN_TO_ARAB.put('X', 10);
        Solution.ROMAN_TO_ARAB.put('V', 5);
        Solution.ROMAN_TO_ARAB.put('I', 1);
    }

    public int romanToInt(String s) {
        if (null == s || s.length() == 0)
            throw new IllegalArgumentException("s is empty");
        int result = 0;
        for(int i = 0; i < s.length() - 1; i++) {
            if(Solution.ROMAN_TO_ARAB.get(s.charAt(i)) < Solution.ROMAN_TO_ARAB.get(s.charAt(i + 1)))
                result -= Solution.ROMAN_TO_ARAB.get(s.charAt(i));
            else
                result += Solution.ROMAN_TO_ARAB.get(s.charAt(i));
        }
        result += Solution.ROMAN_TO_ARAB.get(s.charAt(s.length() - 1));
        return result;
    }
}
```