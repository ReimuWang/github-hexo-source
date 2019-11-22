---
title: Java 算法题-翻转字符串
date: 2017-09-28 17:02:36
tags: [算法,Java]
categories: Java 算法题
---

# 使用系统方法

<!-- more -->

```
public static String reverse(String str) {
    if(null == str || str.length() <= 1) return str;
    return new StringBuilder(str).reverse().toString();
}
```

# 递归

```
public static String reverse(String str) {
    if(null == str || str.length() <= 1) return str;
    return reverse(str.substring(1)) + str.charAt(0);
}
```

# 转为char数组

```
public static String reverse(String str) {
    if (null == str || str.length() <= 1) return str;
    char[] strArr = str.toCharArray();
    int begin = 0, end = strArr.length - 1;
    while (begin < end) {
        char temp = strArr[begin];
        strArr[begin] = strArr[end];
        strArr[end] = temp;
        begin++;
        end--;
    }
    return new String(strArr);
}
```