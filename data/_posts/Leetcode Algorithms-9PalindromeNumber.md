---
title: Leetcode Algorithms-9.Palindrome Number
date: 2017-05-12 00:41:39
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Palindrome Number | LeetCode OJ](https://leetcode.com/problems/palindrome-number/#/description)

# 问题描述

![0.png](/images/blog_pic/Leetcode Algorithms/9PalindromeNumber/0.png)

<!-- more -->

# 解法

**思路**

将原始数字翻转后与原始数字比较，若相等即为回文数。

原始数据除到个位数时即停止计算，此时翻转后的数字比原数少一位，因此必然不会越界。

**注意**

1. 不使用额外空间是不可能的，题意应该是指空间复杂度应为O(1)。
2. 负数不是回文数。

**代码**

```
public class Solution {

    public boolean isPalindrome(int x) {
        if (x < 0) return false;
        int originalReduce = x;
        int reverseIncrease = 0;
        int measure = 10;
        while (originalReduce >= measure) {
            reverseIncrease = reverseIncrease * measure + originalReduce % measure;
            originalReduce /= measure;
        }
        return reverseIncrease == x / measure && originalReduce == x % measure;
    }
}
```

以上是leetcode中给出的官方解法，它从根本上避免了int型的越界。不过在我来看，如果用的语言是Java的话，即便越界了也没有什么。原始的x不可能是一个越界的数，如果它在翻转后越界了，说明它就不可能是一个回文数。因此我们可以不用考虑越界问题，这样写：

```
class Solution {

    public boolean isPalindrome(int x) {
        if (x < 0) return false;
        int result = 0;
        int base = x;
        int m = 10;
        while (x != 0) {
            result = m * result + x % m;
            x /= m;
        }
        return result == base;
    }
}
```

事实上，这种写法确实也能通过leetcode的检查。