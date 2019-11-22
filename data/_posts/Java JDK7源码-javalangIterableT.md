---
title: Java JDK7源码-java.lang.Iterable&lt;T&gt;
date: 2017-05-23 23:58:36
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.lang;

import java.util.Iterator;

/**
 * Iterable<T>中的T代表Iterator返回的元素类型。
 * 实现本接口的类可用语法糖foreach(即增强for循环)进行遍历，foreach实际是由Iterator实现的。
 */
public interface Iterable<T> {

    Iterator<T> iterator();
}
```

# 已整理层级关系

***直接子接口***

- [java.util.Collection&lt;E&gt;](/2017/05/23/Java JDK7源码-javautilCollectionE/)

# 未整理层级关系

***直接子接口***

- [java.nio.file.DirectoryStream&lt;T&gt;]()
- [java.nio.file.Path]()

***直接实现本接口的类***

- [java.util.ServiceLoader&lt;S&gt;]()
- [java.sql.SQLException]()