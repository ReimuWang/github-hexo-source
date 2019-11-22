---
title: Java 基础-instanceof与isInstance
date: 2017-11-05 13:42:49
tags: [Java,instanceof,isInstance]
categories: Java 基础
---

instanceof与isInstance通常用于规避类型转换异常(ClassCastException)。

<!-- more -->

```
package com.test;

public class Test {
    // obj instanceof(Type):对象的实际类型是否为某类型(子类接口均可)
    // Class.isInstance(obj):该对象的实际类型能否转化为某类(强制自动均可)。Class为类对象
    public static void main(String[] args) {
        A a = new A();
        B b = new B();
        A ba = new B();
        System.out.println("1------------");
        System.out.println(b instanceof B);    // true
        System.out.println(b instanceof A);    // true
        System.out.println(b instanceof Object);    // true
        System.out.println(ba instanceof A);    // true
        System.out.println(ba instanceof B);    // true
        System.out.println(a instanceof B);    // false
        System.out.println(null instanceof Object);    // false
        System.out.println("2------------");
        System.out.println(b.getClass().isInstance(b));    // true
        System.out.println(b.getClass().isInstance(a));    // false
        System.out.println("3------------");
        System.out.println(a.getClass().isInstance(ba));    // true
        System.out.println(b.getClass().isInstance(ba));    // true
        System.out.println(b.getClass().isInstance(null));    // false
        System.out.println("4------------");
        System.out.println(A.class.isInstance(a));    // true
        System.out.println(A.class.isInstance(b));    // true
        System.out.println(A.class.isInstance(ba));    // true
        System.out.println("5------------");
        System.out.println(B.class.isInstance(a));    // false
        System.out.println(B.class.isInstance(b));    // true
        System.out.println(B.class.isInstance(ba));    // true
        System.out.println("6------------");
        System.out.println(Object.class.isInstance(b));    // true
        System.out.println(Object.class.isInstance(null));    // false
    }
}

class A {
}

class B extends A {
}
```