---
title: Java 算法题-查找
date: 2017-10-12 10:37:36
tags: [算法,Java,查找]
categories: Java 算法题
---

# 二分查找

时间复杂度：O(logN)

<!-- more -->

```
public class Search {

    public static int serch(int[] a, int key) {
        if (null == a)
            throw new NullPointerException("a is null");
        return search(a, key, 0, a.length - 1);
    }

    private static int search(int[] a, int key, int begin, int end) {
        if (begin > end) return -1;
        int mid = begin + (end - begin) / 2;
        if (key == a[mid]) return mid;
        if (key < a[mid]) return search(a, key, begin, mid - 1);
        if (key > a[mid]) return search(a, key, mid + 1, end);
        return -1;
    }

    public static void main(String[] args) {
        int[] a = {1, 2, 3, 5};
        System.out.println(serch(a, 3));
    }
}
```