---
title: Leetcode Algorithms-461.Hamming Distance
date: 2017-05-01 13:33:41
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Hamming Distance | LeetCode OJ](https://leetcode.com/problems/hamming-distance/#/description)

# 问题描述

![0.png](/images/blog_pic/Leetcode Algorithms/461HammingDistance/0.png)

<!-- more -->

# 解法1

**思路**

使用语言自带方法输出抑或位个数。

**代码**

```
public class Solution {

    public int hammingDistance(int x, int y) {
        if (x < 0 || y < 0)
            throw new IllegalArgumentException("x < 0 || y < 0");
        return Integer.bitCount(x ^ y);
    }
}
```

# 解法2

**思路**

自行实现输出抑或位个数。

**代码**

```
public class Solution {

    public int hammingDistance(int x, int y) {
        if (x < 0 || y < 0)
            throw new IllegalArgumentException("x < 0 || y < 0");
        int xor = x ^ y;
        int result = 0;
        while(xor != 0) {
            result += xor & 1;
            xor >>= 1;
        }
        return result;
    }
}
```