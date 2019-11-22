---
title: Leetcode Algorithms-566.Reshape the Matrix
date: 2017-05-07 18:44:57
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Reshape the Matrix | LeetCode OJ](https://leetcode.com/problems/reshape-the-matrix/#/description)

# 问题描述

![0.png](/images/blog_pic/Leetcode Algorithms/566ReshapetheMatrix/0.png)

<!-- more -->

# 总体思路

将原始二维数组（不妨设为O）按行读取放入一维数组中(不妨设为T)，再将该一维数组按行填入结果二维数组中（不妨设为R）。

# 解法1

**思路**

遍历O，通过T建立起O,R索引之间的联系。

**代码**

代码路径：

```
public class Solution {

    public int[][] matrixReshape(int[][] nums, int r, int c) {
        if (null == nums)
            throw new NullPointerException("nums is null");
        if (r < 0 || c < 0)
            throw new IllegalArgumentException("r < 0 || c < 0");
        if(nums.length * nums[0].length != r * c)
            return nums;
        int[][] result = new int[r][c];
        int line = 0;
        for(int i = 0; i < nums.length; i++) {
            for(int j = 0; j < nums[0].length; j++) {
                result[line / c][line % c] = nums[i][j];
                line++;
            }
        }
        return result;
    }
}
```

# 解法2

**思路**

遍历O，不通过T，直接建立O,R索引之间的联系。

**代码**

```
public class Solution {

    public int[][] matrixReshape(int[][] nums, int r, int c) {
        if (null == nums)
            throw new NullPointerException("nums is null");
        if (r < 0 || c < 0)
            throw new IllegalArgumentException("r < 0 || c < 0");
        if(nums.length * nums[0].length != r * c)
            return nums;
        int[][] result = new int[r][c];
        int tempRow = 0;
        int tempColumn = 0;
        for(int i = 0; i < nums.length; i++) {
            for(int j = 0; j < nums[0].length; j++) {
                result[tempRow][tempColumn]=nums[i][j];
                tempColumn++;
                if(tempColumn == c) {
                    tempRow++;
                    tempColumn = 0;
                }
            }
        }
        return result;
    }
}
```

# 解法3

**思路**

遍历T，通过T建立起O,R索引之间的联系。

**代码**

```
public class Solution {

    public int[][] matrixReshape(int[][] nums, int r, int c) {
        if (null == nums)
            throw new NullPointerException("nums is null");
        if (r < 0 || c < 0)
            throw new IllegalArgumentException("r < 0 || c < 0");
        int expectTotalLength = r * c;
        if(nums.length * nums[0].length != expectTotalLength)
            return nums;
        int[][] result = new int[r][c];
        for(int i = 0; i < expectTotalLength; i++)
            result[i / c][i % c] = nums[i / nums[0].length][i % nums[0].length];
        return result;
    }
}
```