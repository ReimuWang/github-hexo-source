---
title: Leetcode Algorithms-26.Remove Duplicates from Sorted Array
date: 2018-05-30 18:11:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Remove Duplicates from Sorted Array - LeetCode](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/26Remove DuplicatesfromSortedArray/0.jpg)

<!-- more -->

# 解法

**代码**

```
public class Solution {

    public int removeDuplicates(int[] nums) {
        if (null == nums || nums.length == 0) return 0;
        int lastIndex = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[lastIndex] == nums[i]) continue;
            nums[++lastIndex] = nums[i];
        }
        return lastIndex + 1;
    }
}
```