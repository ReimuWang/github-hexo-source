---
title: Java JDK7源码-java.util.Iterator&lt;E&gt;
date: 2017-05-24 00:01:33
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

import java.util.NoSuchElementException;

public interface Iterator<E> {

    /**
     * 若集合还有元素则返回true，反之返回false。
     * 更清晰的表述为：若iterator游标当前所指向的位置还有元素则返回true，反之返回false。
     * 若在本方法返回false时继续调用next()方法，则next()方法会抛出异常。
     */
    boolean hasNext();

    /**
     * 返回iterator游标当前所指向的元素，随后游标后移一位。
     * 
     * @throws NoSuchElementException iterator游标当前所指向的位置已无元素
     */
    E next();

    /**
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 
     * 移除iterator游标当前所指元素的前一个元素。即上一次调用next()方法所返回的元素。
     * 因此，调用本方法之前必须调用一次next()方法。
     * 调用一次next()方法后多次调用本方法是不允许的。本方法必须与next()方法一一配对。
     * 
     * 这么设计是符合逻辑的，因为我们一般只有先用next()方法得到元素后，才能知道该元素是否需要被移除。
     * 因此移除的是游标的前一个元素，而非当前游标正指向的这个我们不知道的元素。
     * 
     * @throws UnsupportedOperationException 实现类未实现本方法。
     * @throws IllegalStateException 本方法未与next()方法一一配对。
     */
    void remove();
}
```

# 已整理层级关系

***直接子接口***

- [java.util.ListIterator&lt;E&gt;](/2017/05/25/Java JDK7源码-javautilListIteratorE/)

***直接实现本接口的内部类***

- [实例成员内部类java.util.AbstractList&lt;E&gt;.Itr](/2017/06/19/Java JDK7源码-javautilAbstractListE/)
- [实例成员内部类java.util.ArrayList&lt;E&gt;.Itr](/2017/07/06/Java JDK7源码-javautilArrayListE/)
- [实例成员内部类java.util.Vector&lt;E&gt;.Itr](/2017/09/05/Java JDK7源码-javautilVectorE/)

# 综述

本接口是Java集合框架中的一员，用于迭代集合中的元素。Iterator&lt;E&gt;中的E代表该迭代器(iterator)返回的元素类型。

Enumeration接口诞生于JDK1.0，而Iterator接口诞生于JDK1.2，是替代Enumeration的改进版本：即二者实现的功能相同，因此在日常开发中更推荐使用Iterator。Iterator与Enumeration的不同之处可归纳为如下两点：

- 在迭代过程中，Iterator允许删除集合中的元素，而Enumeration则不能
- Iterator使用更为精简易读的方法名

# 未整理层级关系

***直接子接口***

- [javax.xml.stream.XMLEventReader]()
- [com.sun.corba.se.pept.transport.ContactInfoListIterator]()

***直接实现本接口的类***

- [java.util.Scanner]()
- [com.sun.org.apache.xerces.internal.util.XMLAttributesIteratorImpl]()

***直接实现本接口的非public类***

- [javax.imageio.spi.FilterIterator&lt;T&gt;]()
- [javax.imageio.spi.PartialOrderIterator]()
- [com.sun.imageio.plugins.jpeg.ImageTypeIterator]()

***直接实现本接口的内部类***

- [实例成员内部类java.util.LinkedList&lt;E&gt;.DescendingIterator]()
- [实例成员内部类java.util.PriorityQueue&lt;E&gt;.Itr]()
- [实例成员内部类java.util.ArrayDeque&lt;E&gt;.DeqIterator]()
- [实例成员内部类java.util.ArrayDeque&lt;E&gt;.DescendingIterator]()
- [实例成员内部类java.util.HashMap&lt;K, V&gt;.HashIterator&lt;E&gt;]()
- [实例成员内部类java.util.TreeMap&lt;K, V&gt;.PrivateEntryIterator&lt;T&gt;]()
- [实例成员内部类java.util.WeakHashMap&lt;K, V&gt;.HashIterator&lt;T&gt;]()
- [实例成员内部类java.util.IdentityHashMap&lt;K, V&gt;.IdentityHashMapIterator&lt;T&gt;]()
- [实例成员内部类java.util.LinkedHashMap&lt;K, V&gt;.LinkedHashIterator&lt;T&gt;]()
- [实例成员内部类java.util.Hashtable&lt;K, V&gt;.Enumerator&lt;T&gt;]()
- [实例成员内部类java.util.EnumMap&lt;K extends Enum&lt;K&gt;, V&gt;.EnumMapIterator&lt;T&gt;]()
- [实例成员内部类java.util.JumboEnumSet&lt;E extends Enum&lt;E&gt;&gt;.EnumSetIterator&lt;E extends Enum&lt;E&gt;&gt;]()
- [实例成员内部类java.util.RegularEnumSet&lt;E extends Enum&lt;E&gt;&gt;.EnumSetIterator&lt;E extends Enum&lt;E&gt;&gt;]()
- [实例成员内部类java.util.ServiceLoader&lt;S&gt;.LazyIterator]()
- [实例成员内部类java.util.concurrent.ArrayBlockingQueue&lt;E&gt;.Itr]()
- [实例成员内部类java.util.concurrent.ConcurrentLinkedQueue&lt;E&gt;.Itr]()
- [实例成员内部类java.util.concurrent.LinkedBlockingQueue&lt;E&gt;.Itr]()
- [实例成员内部类java.util.concurrent.LinkedTransferQueue&lt;E&gt;.Itr]()
- [实例成员内部类java.util.concurrent.PriorityBlockingQueue&lt;E&gt;.Itr]()
- [实例成员内部类java.util.concurrent.DelayQueue&lt;E extends Delayed&gt;.Itr]()
- [实例成员内部类java.util.concurrent.ConcurrentLinkedDeque&lt;E&gt;.AbstractItr]()
- [实例成员内部类java.util.concurrent.LinkedBlockingDeque&lt;E&gt;.AbstractItr]()
- [实例成员内部类java.util.concurrent.ConcurrentHashMap&lt;K, V&gt;.EntryIterator]()
- [实例成员内部类java.util.concurrent.ConcurrentHashMap&lt;K, V&gt;.KeyIterator]()
- [实例成员内部类java.util.concurrent.ConcurrentHashMap&lt;K, V&gt;.ValueIterator]()
- [实例成员内部类java.util.concurrent.ConcurrentSkipListMap&lt;K, V&gt;.Iter&lt;T&gt;]()
- [实例成员内部类javax.xml.soap.MimeHeaders.MatchingIterator]()
- [实例成员内部类javax.print.attribute.standard.PrinterStateReasons.PrinterStateReasonSetIterator]()
- [实例成员内部类com.sun.corba.se.impl.encoding.BufferManagerWriteCollect.BufferManagerWriteCollectIterator]()
- [实例成员内部类com.sun.org.apache.xerces.internal.util.NamespaceSupport.IteratorPrefixes]()
- [非public类-实例成员内部类javax.print.MimeType.ParameterMapEntrySetIterator]()
- [静态成员内部类java.util.Collections.EmptyIterator&lt;E&gt;]()
- [静态成员内部类java.beans.beancontext.BeanContextSupport.BCSIterator]()
- [静态成员内部类javax.imageio.ImageIO.ImageReaderIterator]()
- [静态成员内部类javax.imageio.ImageIO.ImageTranscoderIterator]()
- [静态成员内部类javax.imageio.ImageIO.ImageWriterIterator]()
- [静态成员内部类javax.xml.validation.SchemaFactoryFinder.SingleIterator]()
- [静态成员内部类javax.xml.xpath.XPathFactoryFinder.SingleIterator]()
- [静态成员内部类com.sun.org.apache.xml.internal.security.keys.storage.implementations.CertsInFilesystemDirectoryResolver.FilesystemIterator]()
- [静态成员内部类com.sun.org.apache.xml.internal.security.keys.storage.implementations.SingleCertificateResolver.InternalIterator]()
- [静态成员内部类com.sun.org.apache.xml.internal.security.keys.storage.implementations.KeyStoreResolver.KeyStoreIterator]()
- [静态成员内部类com.sun.org.apache.xml.internal.security.keys.keyresolver.KeyResolver.ResolverIterator]()
- [静态成员内部类com.sun.org.apache.xml.internal.security.keys.storage.StorageResolver.StorageResolverIterator]()
- [静态成员内部类-实例成员内部类java.util.TreeMap&lt;K, V&gt;.NavigableSubMap&lt;K, V&gt;.SubMapIterator&lt;T&gt;]()
- [静态成员内部类-实例成员内部类java.util.concurrent.ScheduledThreadPoolExecutor.DelayedWorkQueue.Itr]()
- [静态成员内部类-实例成员内部类java.util.concurrent.ConcurrentSkipListMap&lt;K, V&gt;.SubMap&lt;K, V&gt;.SubMapIter&lt;T&gt;]()