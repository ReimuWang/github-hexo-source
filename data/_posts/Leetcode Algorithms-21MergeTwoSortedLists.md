---
title: Leetcode Algorithms-21.Merge Two Sorted Lists
date: 2018-05-30 18:11:43
tags: [Leetcode,Algorithms,Java]
categories: Leetcode Algorithms
---

# 问题地址

[Merge Two Sorted Lists - LeetCode](https://leetcode.com/problems/merge-two-sorted-lists/description/)

# 问题描述

![0.jpg](/images/blog_pic/Leetcode Algorithms/21MergeTwoSortedLists/0.jpg)

<!-- more -->

# 解法

**思路**

注意在迭代的过程中不要将链表弄断即可。

**代码**

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public class Solution {

    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode l1Current = l1;
        ListNode l2Current = l2;
        if (null == l1Current) return l2Current;
        if (null == l2Current) return l1Current;
        ListNode head = null;
        if (l1Current.val < l2Current.val) {
            head = l1Current;
            l1Current = l1Current.next;
        } else {
            head = l2Current;
            l2Current = l2Current.next;
        }
        ListNode current = head;
        while (null != l1Current && null != l2Current) {
            if (l1Current.val < l2Current.val) {
                current.next = l1Current;
                l1Current = l1Current.next;
            } else {
                current.next = l2Current;
                l2Current = l2Current.next;
            }
            current = current.next;
        }
        if (null == l1Current) current.next = l2Current;
        if (null == l2Current) current.next = l1Current;
        return head;
    }
}

class ListNode {

    int val;

    ListNode next;

    ListNode(int x) {
        val = x;
    }
}
```