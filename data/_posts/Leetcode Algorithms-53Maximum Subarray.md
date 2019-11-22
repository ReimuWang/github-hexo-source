---
title: Leetcode Algorithms-53.Maximum Subarray
date: 2018-09-17 11:04:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Maximum Subarray - LeetCode](https://leetcode.com/problems/maximum-subarray/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/53Maximum Subarray/0.jpg)

<!-- more -->

# 解法1

最为笨拙的暴力穷举解法，时间复杂度O(n2)。

**代码**

```
public class Solution {

    public int maxSubArray(int[] nums) {
        if (null == nums || nums.length == 0) return 0;
        int result = nums[0];
        for (int i = 0; i < nums.length; i++) {
            int temp = 0;
            for (int j = i; j < nums.length; j++) {
                temp += nums[j];
                if (temp > result) result = temp;
            }
        }
        return result;
    }
}
```

# 解法2

这是一个时间复杂度为O(n)解法。

定义3个变量：

- max:当前已找到的最大和。遍历结束后返回的即是该值。
- now:当前求和的起始元素。这个值并未在代码中显式定义，而是隐含在sum中。
- sum:当前求得的和。

我们不妨先将问题分为两类：全为负数的情况及并非全为负数的情况。

当数组中的元素全部为负数时，此时将元素相加毫无意义(因为只会越加越小)，此时问题将退化为筛选出数组中最大的元素。即每次循环均有：

```
sum=nums[i]
```

然后和max比较以筛选出最大值。

当数组中的元素并非全为负数时，一旦某次加法进行后导致和变为了负数，说明此次加法操作肯定是不应该的：毕竟本情况下数组中是有非负数的，单拎出来最大值也不会是负数。此时就应更换起点，然后和历史上得到的最大值做比较。依次类推，最终返回最大值。

**代码**

```
public class Solution {

    public int maxSubArray(int[] nums) {
        if (null == nums || nums.length == 0) return 0;
        int max = Integer.MIN_VALUE, sum = 0;
        for (int i = 0; i < nums.length; i++) {
            sum = sum < 0 ? nums[i] : sum + nums[i];
            if (sum > max) max = sum;
        }
        return max;
    }
}
```