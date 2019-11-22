---
title: Leetcode Algorithms-70.Climbing Stairs
date: 2018-09-17 18:37:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Climbing Stairs - LeetCode](https://leetcode.com/problems/climbing-stairs/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/70Climbing Stairs/0.jpg)

<!-- more -->

# 解法

本题实际上就是一个斐波那契数列的变种，原始的斐波那契数列的定义为：

```
F(0)=0
F(1)=1
F(n)=F(n-1)+F(n-2) (n>=2)
```

即是一组这样的数:0,1,1,2,3,5,8...

本体并未改变斐波那契数列的根本特性(后一项是前两项的和)，只是拿掉了最初的两项，同时n也从1开始了：

```
F(1)=1
F(2)=2
F(n)=F(n-1)+F(n-2) (n>=3)
```

即是一组这样的数:1,2,3,5,8...

从最容易理解的角度来看，我们可以写出这样的递归代码：

```
public class Solution {

    public int climbStairs(int n) {
        return n <= 2 ? n : climbStairs(n - 2) + climbStairs(n - 1);
    }
}
```

不过，这种解法的时间复杂度将达到恐怖的2^n。因为我们欲得到F(n)，则必须先得到F(n-1)及F(n-2)，依此类推。

因此，我们通常会采用如下时间复杂度为O(n)的解法：

```
public class Solution {

    public int climbStairs(int n) {
        if (n <= 2) return n;
        int first = 1;
        int second = 2;
        int now = 0;
        for (int i = 3; i <= n; i++) {
            now = first + second;
            first = second;
            second = now;
        }
        return now;
    }
}
```