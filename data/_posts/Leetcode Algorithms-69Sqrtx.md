---
title: Leetcode Algorithms-69.Sqrt(x)
date: 2018-09-17 18:02:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Sqrt(x) - LeetCode](https://leetcode.com/problems/sqrtx/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/69Sqrtx/0.jpg)

<!-- more -->

# 解法

```
public class Solution {

    public int mySqrt(int x) {
        if (x <= 1) return x;
        int begin = 0;
        int end = x;
        while (begin <= end) {
            int mid = begin + (end - begin) / 2;
            int now = x / mid;
            if (now == mid) return mid;
            else if (now > mid) {
                if (mid + 1 > x / (mid + 1)) return mid;
                else begin = mid + 1;
            } else end = mid - 1;
        }
        return -1;
    }
}
```

显然，出题人不希望我们直接使用Math.sqrt()。

本解法是二分查找的变种，需要注意的是解法中使用了形如：

```
int now = x / mid;
```

的代码，其实更符合逻辑的思路应该是正向相乘：

```
int now = mid * mid;
```

不过，这样会有溢出的风险，因此还是应该用题中除法的形式。

另外，0是不能做除数的，因此一开始的边界判断也一定要做好。