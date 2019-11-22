---
title: Leetcode Algorithms-561.Array Partition 1
date: 2017-04-26 22:06:05
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Array Partition I | LeetCode OJ](https://leetcode.com/problems/array-partition-i/#/description)

# 问题描述

![0.png](/images/blog_pic/Leetcode Algorithms/561ArrayPartition1/0.png)

<!-- more -->

# 解法1

**思路**

排序后输出奇数项。

- 时间复杂度：取决于选用的排序算法的时间复杂度；除此之外的时间复杂度为O(n)
- 空间复杂度：取决于选用的排序算法的空间复杂度；除此之外的空间复杂度为O(1)

**代码**

```
import java.util.Arrays;

public class Solution {

    public int arrayPairSum(int[] nums) {
        if (null == nums)
            throw new NullPointerException("nums is null");
        if (nums.length < 2 || nums.length > 20000)
            throw new IllegalArgumentException("nums 's length < 2 || > 20000");
        int result = 0;
        Arrays.sort(nums);
        for(int i = 0; i < nums.length; i = i + 2)
            result +=  nums[i];
        return result;
    }
}
```

# 解法2

**思路**

题设中规定n为整数，且取值范围为[-10000, 10000]。则可声明一个长度为20001的数组，不妨设为hashArray，该数组作为散列表即可存储所有2n个整数。解决冲突的方法为自增1。即hashArray中元素的值代表其索引值在原2n个整数中出现的次数。hashArray中所有元素求和即为2n。

hashArray生成完成后，其实就完成了对原数组的排序。且时间复杂度为O(n)，空间复杂度O(n)。

遍历hashArray，设当前进行到的下标为i，则hashArray[i]按奇偶数及输出状态将情况分为以下4种：

|    |输出                                                                  |不输出                                                          |
|:---|:--------------------------------------------------------------------:|:--------------------------------------------------------------:|
|奇数|值：(i - 10000) * (hashArray[i] + 1) / 2<br>下一个索引开始状态：不输出|值：(i - 10000) * (hashArray[i] / 2)<br>下一个索引开始状态：输出|
|偶数|值：(i - 10000) * hashArray[i] / 2<br>下一个索引开始状态：输出        |值：(i - 10000) * hashArray[i] / 2<br>下一个索引开始状态：不输出|

其中

- **输出/不输出** 是指：因只需输出有序数组中的奇数项，故某下标开始时会有一个输出或不输出的状态；结束时又会为下一个下标生成一个输出或不输出的状态。
- **奇数/偶数** 是指：该下标中存储的值，即为下标值在原始数据中出现的次数。
- 原始数据 --> hashArray 做了 [-10000, 10000] --> [0, 20000] 的值映射，因此实际参与计算时应还原回原值。

**代码**

奇数-不输出时，(i - 10000) * (hashArray[i] / 2)中，(hashArray[i] / 2)的括号一定要有，保证整数下取整机制。

```
public class Solution {

    public int arrayPairSum(int[] nums) {
        if (null == nums)
            throw new NullPointerException("nums is null");
        if (nums.length < 2 || nums.length > 20000)
            throw new IllegalArgumentException("nums 's length < 2 || > 20000");
        int limit = 10000;
        int result = 0;
        int[] hashArray = new int[2 * limit + 1];
        boolean ifAdd = true;
        for(int num : nums) hashArray[num + limit]++;
        for(int i = 0; i < hashArray.length; i++) {
            if(hashArray[i] % 2 == 0) {
                result = result + (i - limit) * hashArray[i] / 2;
            } else {
                if(ifAdd)
                    result = result + (i - limit) * (hashArray[i] + 1) / 2;
                else
                    result = result + (i - limit) * (hashArray[i] / 2);
                ifAdd = !ifAdd;
            }
        }
        return result;
    }
}
```