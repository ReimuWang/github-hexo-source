---
title: Leetcode Algorithms-1.Two Sum 
date: 2017-05-07 22:36:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Two Sum | LeetCode OJ](https://leetcode.com/problems/two-sum/#/description)

# 问题描述

![0.png](/images/blog_pic/Leetcode Algorithms/1TwoSum/0.png)

<!-- more -->

# 解法

**思路**

使用哈希表存储已遍历过的数字。

**代码**

```
import java.util.HashMap;
import java.util.Map;

public class Solution {

    public int[] twoSum(int[] nums, int target) {
        if (null == nums)
            throw new NullPointerException("nums is null");
        int[] result = new int[2];
        Map<Integer, Integer> numberMap = new HashMap<Integer, Integer>();
        for(int i = 0; i < nums.length; i++) {
            int assume = target - nums[i];
            if(numberMap.containsKey(assume)) {
                result[0] =  numberMap.get(assume);
                result[1] = i;
                break;
            }
            numberMap.put(nums[i], i);
        }
        return result;
    }
}
```