---
title: Leetcode Algorithms-7.Reverse Integer
date: 2017-05-08 00:05:00
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Reverse Integer | LeetCode OJ](https://leetcode.com/problems/reverse-integer/#/description)

# 问题描述

![0.png](/images/blog_pic/Leetcode Algorithms/7ReverseInteger/0.png)

<!-- more -->

# 解法1

**思路**

使用long型避免溢出。

**注意**

1. Java中int型取值范围为[-2^31 ,2^31 -1]。
2. Java中a%b=a-(a/b)*b

**代码**

```
public class Solution {

    public int reverse(int x) {
        int measure = 10;
        long result = 0;
        while(x != 0) {
            result = measure * result + x % measure;
            if(result < Integer.MIN_VALUE || result > Integer.MAX_VALUE)
                return 0;
            x /= measure;
        }
        return (int)result;
    }
}
```

# 解法2

**思路**

每次计算后将算得的结果还原回去，若与原值不等则越界。无需使用long型避免越界。

**代码**

代码路径：

```
public class Solution {

    public int reverse(int x) {
        int result = 0;
        int m = 10;
        while (x >= m || x <= -1 * m) {
            result = m * result + x % m;
            x /= m;
        }
        int tempResult = m * result + x;
        if ((tempResult - x) / m != result)
            return 0;
        else
            result = tempResult;
        return result;
    }
}
```