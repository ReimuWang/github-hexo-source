---
title: Java JDK7源码-java.util.ListIterator&lt;E&gt;
date: 2017-05-25 21:29:22
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public interface ListIterator<E> extends Iterator<E> {

    /**
     * 查询操作
     * 继承父接口:java.util.Iterator<E>
     * 当向后遍历还有元素时返回true。
     * 换句话说，返回ture说明仍然可以通过next()取到元素。
     * 若在本方法返回false时继续调用next()，则next()会抛出异常。
     */
    boolean hasNext();

    /**
     * 查询操作
     * 继承父接口:java.util.Iterator<E>
     * 返回listIterator当前游标后一个元素，同时游标后移一位。
     * 在迭代list的过程中本方法可能会被反复调用，或者与previous()交替被调用。以此控制向前或向后迭代。
     * 
     * 连续执行的情况下，以下语句均会返回当前游标的后一个元素：
     * listIterator.next();
     * listIterator.previous();
     * 
     * 连续执行的情况下，以下语句均会返回当前游标的前一个元素：
     * listIterator.previous();
     * listIterator.next();
     * 
     * @throws NoSuchElementException 游标所指的位置之后已无元素。
     */
    E next();

    /**
     * 查询操作
     * 当向前遍历还有元素时返回true。
     * 换句话说，返回ture说明仍然可以通过previous()取到元素。
     * 若在本方法返回false时继续调用previous()，则previous()会抛出异常。
     */
    boolean hasPrevious();

    /**
     * 查询操作
     * 返回listIterator当前游标前一个元素，同时游标前移一位。
     * 在迭代list的过程中本方法可能会被反复调用，或者与next()交替被调用。以此控制向前或向后迭代。
     * 
     * 连续执行的情况下，以下语句均会返回当前游标的后一个元素：
     * listIterator.next();
     * listIterator.previous();
     * 
     * 连续执行的情况下，以下语句均会返回当前游标的前一个元素：
     * listIterator.previous();
     * listIterator.next();
     * 
     * @throws NoSuchElementException 游标所指的位置之前已无元素。
     */
    E previous();

    /**
     * 查询操作
     * 返回listIterator当前游标后一个元素的索引。
     * 本操作不会移动游标。即本方法会返回下次调用next()时返回的元素的索引。
     * 特别的，当游标位于list末尾时，调用本方法不会抛出异常，而是会返回list的长度。
     */
    int nextIndex();

    /**
     * 查询操作
     * 返回listIterator当前游标前一个元素的索引。
     * 本操作不会移动游标。即本方法会返回下次调用previous()时返回的元素的索引。
     * 特别的，当游标位于list最左边时，调用本方法不会抛出异常，而是会返回-1。
     */
    int previousIndex();

    /**
     * 改变操作
     * 继承父接口:java.util.Iterator<E>
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 
     * 移除上一次由next()或previous()返回的元素。
     * 本方法必须与next()或previous()一一配对。且每个配对之间不能调用add(E e)。
     * 
     * @throws UnsupportedOperationException listIterator不支持本方法。
     * @throws IllegalStateException：本方法未与next()或previous()一一配对，或配对之间调用了add(E e)。
     */
    void remove();

    /**
     * 改变操作
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 
     * 用e覆盖上一次由next()或previous()返回的元素的位置的元素。
     * 本方法在调用前必须调用一次next()或previous()，否则不知道该覆盖哪个位置的元素。
     * 且在本方法与最后一次next()或previous()之间不能调用add(E e)或remove()。
     * 
     * @throws UnsupportedOperationException listIterator不支持本方法。
     * @throws ClassCastException e因为其所属的类禁止被插入list。
     * @throws IllegalArgumentException e因其某些属性禁止被插入list。
     * @throws IllegalStateException 本方法在调用前没有调用next()或previous()，或在本方法与最后一次next()或previous()之间调用add(E e)或remove()。
     */
    void set(E e);

    /**
     * 改变操作
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 
     * 将e插入当前游标所在位置。
     * 若list为空，则e将成为list的第一个元素。
     * e插入后，当前游标位于e之后的位置。
     * 因此插入后第一次如果调用的是next()将返回插入前游标所指的下一个元素；第一次如果调用的是previous()将返回e。
     * 
     * 调用该方法后，若调用nextIndex()或previousIndex()，和未调用该方法之前相比值均会增加1。
     * 
     * @throws UnsupportedOperationException listIterator不支持本方法。
     * @throws ClassCastException e因为其所属的类禁止被插入list。
     * @throws IllegalArgumentException e因其某些属性禁止被插入list。
     */
    void add(E e);
}
```

# 已整理层级关系

***直接父接口***

- [java.util.Iterator&lt;E&gt;](/2017/05/24/Java JDK7源码-javautilIteratorE/)

# 综述

本接口是Java集合框架中的一员。是专为List接口设计的迭代器。可以双向迭代并修改list，并可获得迭代器当前的游标位置。

与父接口Iterator不同的是，ListIterator没有当前元素的概念。它的游标位置在两个元素之间。当调用previous()时将返回紧邻游标之前的一个元素同时游标前移一位；当调用next()时将返回紧邻游标之后的一个元素同时游标后移一位。

若list的长度为n，则有n+1个位置可供放置游标。形如：

```
                     Element(0)   Element(1)   Element(2)   ... Element(n-1)
cursor positions:  ^            ^            ^            ^                  ^
```

游标无法定义remove()及set(E e)。因为这两个方法操作的是元素本身而游标指向的是元素之间的位置。这两个方法必须与next()或previous()配对(remove()要求一一配对而set(E e)不需要)，其操作的元素即为配对方法返回的元素。