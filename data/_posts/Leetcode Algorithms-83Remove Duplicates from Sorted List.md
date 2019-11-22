---
title: Leetcode Algorithms-83.Remove Duplicates from Sorted List
date: 2018-09-17 19:11:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Remove Duplicates from Sorted List - LeetCode](https://leetcode.com/problems/remove-duplicates-from-sorted-list/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/83Remove Duplicates from Sorted List/0.jpg)

<!-- more -->

# 解法

非递归：

```
public class Solution {

    public ListNode deleteDuplicates(ListNode head) {
        if (null == head) return head;
        ListNode tail = head;
        ListNode now = head.next;
        while (null != now) {
            if (tail.val != now.val) {
                tail.next = now;
                tail = now;
            }
            now = now.next;
        }
        tail.next = null;
        return head;
    }
}

class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}
```

递归：

```
public class Solution {

    public ListNode deleteDuplicates(ListNode head) {
        if(null == head || null == head.next) return head;
        head.next = deleteDuplicates(head.next);
        return head.val == head.next.val ? head.next : head;
    }
}

class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}
```