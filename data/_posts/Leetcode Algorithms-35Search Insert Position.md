---
title: Leetcode Algorithms-35.Search Insert Position
date: 2018-09-14 17:52:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Search Insert Position - LeetCode](https://leetcode.com/problems/search-insert-position/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/35Search Insert Position/0.jpg)

<!-- more -->

# 解法

```
public class Solution {

    public int searchInsert(int[] nums, int target) {
        if (null == nums) return -1;
        int low = 0;
        int high = nums.length - 1;
        while (low <= high) {
            int mid = low + (high - low) / 2;
            if (target == nums[mid]) return mid;
            if (target < nums[mid]) high = mid - 1;
            else low = mid + 1;
        }
        return low;
    }
}
```

本问题是二分查找的变种。当target在nums中时，和普通的二分查找一样返回mid即可。需要说明的是，当没有找到时，为何返回了low。

当无法找到时，最后一次查找必在相邻两个元素之间进行，即必有

```
high=low+1
mid=low+(high-low)/2=low
```

本次若是找到，即得出mid=target，那是最好。此时直接返回mid即可。若是找不到，在target<mid时应返回low，而target>mid时应返回high。我们可以回看代码，第一种情况下比较后low没有改变；而第二种情况下比较后low=mid+1，即low=low+1，恰好等于需要的high了。因此在最后一次比较结束后，直接返回low即可。