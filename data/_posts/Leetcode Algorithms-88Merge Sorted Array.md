---
title: Leetcode Algorithms-88.Merge Sorted Array
date: 2018-09-18 10:56:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Merge Sorted Array - LeetCode](https://leetcode.com/problems/merge-sorted-array/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/88Merge Sorted Array/0.jpg)

<!-- more -->

# 解法1

正向迭代，此时需要顾虑nums1的长度变化情况，因此比较臃肿。

```
public class Solution {

    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i1 = 0;
        int lengthI1 = m; 
        int orginI1 = 0;
        int i2 = 0;
        while (orginI1 < m && i2 < n) {
            if (nums2[i2] >= nums1[i1]) orginI1++;
            else {
                this.insert(nums2[i2], nums1, lengthI1, i1);
                lengthI1++;
                i2++;
            }
            i1++;
        }
        if (orginI1 == m)
            for (int i = i2; i < n; i++) nums1[i1++] = nums2[i];
    }

    private void insert(int value, int[] nums, int length, int index) {
        for (int i = length; i > index; i--) nums[i] = nums[i - 1];
        nums[index] = value;
    }
}
```

# 解法2

逆向迭代，看起来好多了。不过时间复杂度其实差不多。

```
public class Solution {

    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i1 = m - 1;
        int i2 = n - 1;
        int now = m + n - 1;
        while (i1 > -1 && i2 > -1)
            nums1[now--] = (nums1[i1] > nums2[i2]) ? nums1[i1--] : nums2[i2--];
        while (i2 > -1) nums1[now--] = nums2[i2--];
    }
}
```