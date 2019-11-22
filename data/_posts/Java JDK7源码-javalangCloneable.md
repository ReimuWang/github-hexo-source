---
title: Java JDK7源码-java.lang.Cloneable
date: 2017-07-04 17:05:36
tags: [Java,源码]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.lang;

public interface Cloneable {
}
```

# 已整理层级关系

***直接实现本接口的类***

- [java.util.ArrayList&lt;E&gt;](/2017/07/06/Java JDK7源码-javautilArrayListE/)
- [java.util.HashMap&lt;K,V&gt;](/2018/11/09/Java JDK7源码-javautilHashMapKV/)

# 综述

一个类如果想进行克隆操作则必须实现本接口(若未实现本接口又强行调用clone()，则可通过编译，但是运行时会抛出CloneNotSupportedException)。

通常来说，实现本接口的类需要重写Object的clone()方法。Object.clone()为protected方法，重写后的方法为public方法。当然，也可以选择不重写，但此时clone()仅有protected权限。

默认o.clone()方法采用浅拷贝，会将o copy一份并赋予新的引用。o中的成员变量若为基本数据类型，无论是浅拷贝还是深拷贝，都会进行原值克隆。毕竟他们都不是对象，不是存储在堆中。注意：基本数据类型并不包括他们对应的包装类。但是若o中的成员变量是一个对象，则浅拷贝不会拷贝该对象，即新拷贝出的o的克隆体中的对象仍为原对象。而深拷贝则会有N层拷贝的概念，即控制实际拷贝到第几层级。

注意本接口未包含clone()。因此仅仅是继承本接口是无法拷贝对象的。即使clone()被反射调用，也无法保证它会成功。