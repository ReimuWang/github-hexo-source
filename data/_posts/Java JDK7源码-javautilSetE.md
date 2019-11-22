---
title: Java JDK7源码-java.util.Set&lt;E&gt;
date: 2017-06-13 11:31:07
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public interface Set<E> extends Collection<E> {

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 返回set包含的元素个数(即set的势)。
     * 若set包含的元素个数大于Integer.MAX_VALUE，则返回Integer.MAX_VALUE。
     */
    int size();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 若set不包含元素则返回true，反之返回false。
     */
    boolean isEmpty();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 若set包含o则返回true。
     * 更具体的说，当且仅当set包含至少一个元素e，满足(o==null ? e==null : o.equals(e))，返回true
     * 
     * 本方法与后文介绍的containsAll(Collection<?> c)方法很相似，可对比学习。
     * 
     * @throws ClassCastException o的类型与set不相容。
     * @throws NullPointerException o==null且set不允许空元素存在。
     */
    boolean contains(Object o);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 返回一个set的迭代器，该迭代器默认从头开始迭代。
     * 通常情况下该迭代器不保证数据有序。
     */
    Iterator<E> iterator();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 本方法是连接集合与数组之间的桥梁。
     * 返回包含set所有元素的数组。若set是有序的(即迭代器返回的结果有序)，则返回的数组必须有同样的顺序。
     * 返回的数组与set之间不存在引用关系(即使set的底层就是基于数组实现的)，数组中的元素是set中元素的浅拷贝。
     * 
     * 本方法与后文介绍的toArray(T[] a)方法很相似，可对比学习。
     */
    Object[] toArray();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 本方法是连接集合与数组之间的桥梁。
     * 
     * 返回包含set所有元素的数组。返回数组的类型即为a的类型。若set是有序的(即迭代器返回的结果有序)，则返回的数组必须有同样的顺序。
     * 具体规则的伪代码为：
     * if (a.length < set.size())
     *     不修改a，而是以a的类型新建长度为set.size()的数组并填入set中的值，随后返回这个新生成的数组。
     * else if (a.length == set.size()
     *     将返回的结果直接填入a后返回a(若a中已有值，则a中的原值会被覆盖)。
     * else
     *     数组索引在[0, set.size() -1]的元素会被set对应位置的元素覆盖。
     *     数组索引 == set.size()的元素会被置为null
     *     数组中后续元素(如果有的话)不变。
     * 
     * 若set不允许包含空元素，则本方法此时可用来计算set的长度：返回的数组第一次出现null的索引即为set的长度。
     * 
     * 返回的数组与set之间不存在引用关系(即使set的底层就是基于数组实现的)，数组中的元素是set中元素的浅拷贝。
     * 
     * 小例子：
     * String[] y = x.toArray(new String[0]);
     * x是一个元素类型为String的set，则上述语句会将x中的元素依序复制一份浅拷贝到数组y。
     * 
     * 关于本方法与前文介绍的toArray()方法，需明确：
     * set.toArray()
     * set.toArray(new Object[s])
     * 当s < set.size()时，上述两行代码的效果等价。
     * 
     * 本方法与前文介绍的toArray()方法很相似，可对比学习。
     * 
     * @throws ArrayStoreException a的类型set不支持。
     * @throws NullPointerException null==a。
     */
    <T> T[] toArray(T[] a);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 修改操作。
     * 本方法属于破坏性方法，可选。
     * 
     * 若e在set中不存在则将e插入到set中。
     * 更一般的说，若set不包含e2这种元素：(e==null ? e2==null : e.equals(e2))，则将e插入set。
     * 若set已包含e，本方法不会改变set并返回false。本方法配合set的构造函数，可保证set永远不会包括重复元素。
     * 以上的规定不意味着在满足不重复的前提下set必须接受所有元素，set可能会拒绝添加任意特定元素，包括null，并抛出一个异常。
     * 
     * 本方法与后文介绍的addAll(Collection<? extends E> c)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException set不支持本方法。
     * @throws ClassCastException e因其类型禁止被插入set。
     * @throws NullPointerException e==null且set禁止包含null。
     * @throws IllegalArgumentException e因其某些属性禁止被插入set。
     */
    boolean add(E e);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 修改操作。
     * 本方法属于破坏性方法，可选。
     * 
     * 移除set中符合条件的元素e：(o==null ? e==null : o.equals(e))。
     * 若本方法改变了set则返回true，反之返回false。
     * 一旦调用本方法后，set将不会再包含与o相等的元素。
     * 
     * 本方法与后文介绍的removeAll(Collection<?> c)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException set不支持本方法。
     * @throws ClassCastException o的类型与set不相容。
     * @throws NullPointerException o==null且set不允许空元素存在。
     */
    boolean remove(Object o);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量查询操作。
     * 
     * 若set包含c中所有元素则返回true，反之返回false。
     * 特别的，若c同样为Set且为set的子集，则返回true。
     *
     * 本方法与前文介绍的contains(Object o)方法很相似，可对比学习。
     * 
     * @throws ClassCastException c中至少有一个元素的类型与set不相容。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且set不允许空元素存在。
     */
    boolean containsAll(Collection<?> c);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 在保证set中不存在重复元素的前提下，将c中所有元素插入set。
     * 特别的，若c同样为一个Set，本方法实际上是将set改造为了该set与c的并集。
     * 未定义发生如下事件时的应对策略：在操作过程中c发生变化。
     * 若set因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法与前文介绍的add(E e)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException set不支持本方法。
     * @throws ClassCastException c中任意一个元素因其类型禁止被插入set。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且set不允许空元素存在。
     * @throws IllegalArgumentException c中任意一个元素因其某些属性禁止被插入set。
     */
    boolean addAll(Collection<? extends E> c);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 保留set中与c的交集元素。
     * 若set因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法与后文介绍的removeAll(Collection<?> c)方法正相反，可对比学习。
     * 
     * @throws UnsupportedOperationException set不支持本方法。
     * @throws ClassCastException c中至少有一个元素的类型与set不相容。
     * @throws NullPointerException c==null或
     *                              set中至少有一个元素为null且c不允许空元素存在。
     */
    boolean retainAll(Collection<?> c);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 从set中移除所有与c的交集元素(求差集)。
     * 若set因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法与前文介绍的remove(Object o)方法很相似，可对比学习。
     * 本方法与前文介绍的retainAll(Collection<?> c)方法正相反，可对比学习。
     * 
     * @throws UnsupportedOperationException set不支持本方法。
     * @throws ClassCastException c中至少有一个元素的类型与set不相容。
     * @throws NullPointerException c==null或
     *                              set中至少有一个元素为null且c不允许空元素存在。
     */
    boolean removeAll(Collection<?> c);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 清空set。该方法执行完成后set将为空。
     * 
     * @throws UnsupportedOperationException set不支持本方法。
     */
    void clear();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 比较及哈希操作
     * 比较o与set是否相等。
     * 
     * 如下情况判断为相等：
     * o为Set，o与set长度相同，set中有o中所有的元素(或者也可以反过来说，o中有set中所有的元素)。
     * 只要满足该相等条件，Set接口的不同实现之间也可判断为相等。
     */
    boolean equals(Object o);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 比较及哈希操作
     * 返回set的hash code。
     * 
     * set的hash code被定义为set中所有元素的hash code的和。
     * 空元素的hash code被定义为0。
     * 若s1与s2为两个Set，这确保了s1.equals(s2)即有s1.hashCode()==s2.hashCode()。
     * 满足了Java语言规范中两个对象equals(Object o)方法相等即要有hashCode()方法相等的规范。
     */
    int hashCode();
}
```

# 已整理层级关系

***直接父接口***

- [java.util.Collection&lt;E&gt;](/2017/05/23/Java JDK7源码-javautilCollectionE/)

***直接实现本接口的类***

- [java.util.AbstractSet&lt;E&gt;](/2017/06/28/Java JDK7源码-javautilAbstractSetE/)

# 综述

本接口是Java集合框架中的一员，是一种没有重复元素的集合，顾名思义，本接口定义的是数学中Set的概念。Set&lt;E&gt;中的E代表Set中元素的类型。

更具体的说，Set中的元素e1,e2不存在如下情况：e1.equals(e2)为true。因此Set中最多只能有一个元素为null。

---

较之父接口Collection，本接口并未添加新的方法。不过对于很多方法的含义都做了特化。为了便于写文档，因此均会在本接口中重复写一遍。

这些含义上的具体化大致集中于如下3部分：

- 构造函数：经由Set的构造函数构造出的Set必须不能包含重复元素
- add()方法
- equals()方法及hashCode()方法

---

当以容易改变的对象(尤其是这种改变会影响到equals()方法的判定时)作为Set的元素时必须要提高警惕。因为一旦成功被放入Set，再修改元素值导致元素间的重复Set本身是无能为力的。因此Set禁止自包含。

---

不同Set的实现类往往还会附加不同的，更为具体的限制。例如，某些实现类禁止包含空元素，某些实现类会限制其所包含的元素的类型。

试图插入一个不合规的元素会抛出一个运行时异常，例如NullPointerException，ClassCastException。

试图查询一个不合规的元素可能会抛出一个异常，或者仅仅只是返回false。

# 未整理层级关系

***直接子接口***

- [java.util.SortedSet&lt;E&gt;]()
- [com.sun.corba.se.impl.orbutil.graph.Graph]()

***直接实现本接口的类***

- [java.util.HashSet&lt;E&gt;]()
- [java.util.SortedSet&lt;E&gt;]()
- [java.util.LinkedHashSet&lt;E&gt;]()

***直接实现本接口的内部类***

- [静态成员内部类java.util.Collections.CheckedSet&lt;E&gt;]()
- [静态成员内部类java.util.Collections.SetFromMap&lt;E&gt;]()
- [静态成员内部类java.util.Collections.SynchronizedSet&lt;E&gt;]()
- [静态成员内部类java.util.Collections.UnmodifiableSet&lt;E&gt;]()
- [静态成员内部类-静态成员内部类java.util.Collections.CheckedMap&lt;K, V&gt;.CheckedEntrySet&lt;K, V&gt;]()