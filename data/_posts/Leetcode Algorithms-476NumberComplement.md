---
title: Leetcode Algorithms-476.Number Complement
date: 2018-05-24 15:31:39
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Number Complement - LeetCode](https://leetcode.com/problems/number-complement/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/476NumberComplement/0.jpg)

<!-- more -->

# 解法

**思路**

Integer.highestOneBit(i)的作用是求出i的最高位1的位置，然后将它的后续都以0填充。例如5=101，传入本函数后输出100=4。

本思路利用了异或运算的一个特性：任何数异或1都相当于取反。

说是任何数，听起来比较吓人，其实一共也就两个数，我们完全可以穷举：

0^1=1,1^1=0

相当于对原数进行取反。

关于异或的其他特性可详见[数据结构-异或运算](/2018/05/24/数据结构-异或运算/)。

**代码**

```
public class Solution {

    public int findComplement(int num) {
        if (num <= 0)
            throw new IllegalArgumentException("num <= 0");
        return num ^ ((Integer.highestOneBit(num) << 1) - 1);
    }
}
```