---
title: Java 算法题-跳线数组中的最大值
date: 2017-10-11 18:31:36
tags: [算法,Java]
categories: Java 算法题
---

求形如4,5,6,1,2,3中的最大值，可能会包含重复值。

<!-- more -->

```
public static int serchMax(int[] a) {
    if (null == a) throw new NullPointerException("a is null");
    return serchMax(a, 0, a.length - 1);
}

private static int serchMax(int[] a, int begin, int end) {
    if (a[end] > a[begin]) return a[end];
    while (begin < end && a[begin] == a[begin + 1]) begin++;
    if (begin == end) return a[begin];
    while (a[end] == a[end - 1]) end--;
    int mid = begin + (end - begin) / 2;
    if (a[mid] > a[mid + 1]) return a[mid];
    if (a[mid] >= a[begin]) {
        return serchMax(a, mid + 1, end);
    } else {
        return serchMax(a, begin, mid - 1);
    }
}
```