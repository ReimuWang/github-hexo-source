---
title: Java 算法题-翻转字符串中的单词
date: 2017-09-28 17:47:36
tags: [算法,Java]
categories: Java 算法题
---

<!-- more -->

```
public static String reverse(String str) {
    if(null == str || str.length() <= 1) return str;
    char[] strArr = str.toCharArray();
    reverse(strArr, 0, strArr.length - 1);
    int begin = 0;
    for (int i = 1; i < strArr.length; i++) {
        if (strArr[i] == ' ') {
            reverse(strArr, begin, i - 1);
            begin = i + 1;
        }
    }
    return new String(strArr);
}

private static void reverse(char[] strArr, int begin, int end) {
    while(begin < end) {
        char temp = strArr[begin];
        strArr[begin] = strArr[end];
        strArr[end] = temp;
        begin++;
        end--;
    }
}
```