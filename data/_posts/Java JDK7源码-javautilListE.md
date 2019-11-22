---
title: Java JDK7源码-java.util.List&lt;E&gt;
date: 2017-05-25 21:38:33
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public interface List<E> extends Collection<E> {

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 返回列表包含的元素个数。
     * 若列表包含的元素个数大于Integer.MAX_VALUE，则返回Integer.MAX_VALUE。
     */
    int size();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 若列表不包含元素则返回true，反之返回false。
     */
    boolean isEmpty();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 若列表包含o则返回true，反之返回false。
     * 更具体的说，当且仅当列表包含至少一个元素e，满足(o==null ? e==null : o.equals(e))时，返回true。
     * 
     * 本方法与后文介绍的containsAll(Collection<?> c)方法很相似，可对比学习。
     * 
     * @throws ClassCastException o的类型与列表不相容。
     * @throws NullPointerException o==null且列表不允许空元素存在。
     */
    boolean contains(Object o);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 返回一个列表的迭代器，该迭代器默认从头开始迭代。
     * 该迭代器遍历得到的元素有序(与列表的固有顺序相同)。
     * 
     * 后文介绍的listIterator()方法是本方法基于List接口的特化，可对比学习。
     * 后文介绍的listIterator(int index)方法是本方法基于List接口的特化，可对比学习。
     */
    Iterator<E> iterator();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 本方法是连接集合与数组之间的桥梁。
     * 
     * 返回包含列表所有元素的数组。该数组的顺序与列表的固有顺序相同。
     * 返回的数组与列表之间不存在引用关系(即使列表是基于数组的)，数组中的元素是列表中元素的浅拷贝。
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
     * 返回包含列表所有元素的数组。该数组的顺序与列表的固有顺序相同。返回数组的类型即为a的类型。
     * 具体规则的伪代码为：
     * if (a.length < list.size())
     *     不修改a，而是以a的类型新建长度为list.size()的数组并填入列表中的值，随后返回这个新生成的数组。
     * else if (a.length == list.size()
     *     将返回的结果直接填入a后返回a(若a中已有值，则a中的原值会被覆盖)。
     * else
     *     数组索引在[0, list.size() -1]的元素会被列表对应位置的元素覆盖。
     *     数组索引 == list.size()的元素会被置为null
     *     数组中后续元素(如果有的话)不变。
     * 
     * 若列表不允许包含空元素，则本方法此时可用来计算列表的长度：返回的数组第一次出现null的索引即为列表的长度。
     * 
     * 返回的数组与列表之间不存在引用关系(即使列表的底层就是基于数组实现的)，数组中的元素是列表中元素的浅拷贝。
     * 
     * 小例子：
     * String[] y = x.toArray(new String[0]);
     * x是一个元素类型为String的列表，则上述语句会将x中的元素依序复制一份浅拷贝到数组y。
     * 
     * 关于本方法与前文介绍的toArray()方法，需明确：
     * list.toArray()
     * list.toArray(new Object[s])
     * 当s < list.size()时，上述两行代码的效果等价。
     * 
     * 本方法与前文介绍的toArray()方法很相似，可对比学习。
     * 
     * @throws ArrayStoreException a的类型列表不支持。
     * @throws NullPointerException null==a。
     */
    <T> T[] toArray(T[] a);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 修改操作。
     * 本方法属于破坏性方法，可选。
     * 
     * 将e加入列表的末尾。
     * 若列表因本方法而改变则返回true，反之则返回false。
     * 
     * 即便实现类支持本方法，它也可能会做某种限制(禁止插入null，限制可插入的元素类型等)。
     * 
     * 本方法与后文介绍的addAll(Collection<? extends E> c)方法很相似，可对比学习。
     * 本方法与后文介绍的方法addAll(int index, Collection<? extends E> c)很相似，可对比学习。
     * 本方法与后文介绍的方法add(int index, E element)很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws ClassCastException e因其类型禁止被插入列表。
     * @throws NullPointerException e==null且列表禁止包含null。
     * @throws IllegalArgumentException e因其某些属性禁止被插入列表。
     */
    boolean add(E e);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 修改操作。
     * 本方法属于破坏性方法，可选。
     * 
     * 移除o在列表中第一次出现的位置的元素。即：
     * o==null ? get(i)==null : o.equals(get(i))
     * 其中为i满足相等条件的最小索引值。
     * 
     * 若本方法改变了列表则返回true，反之返回false。
     * 
     * 本方法与后文介绍的removeAll(Collection<?> c)方法很相似，可对比学习。
     * 本方法与后文介绍的remove(int index)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws ClassCastException o的类型与列表不相容。
     * @throws NullPointerException o==null且列表不允许空元素存在。
     */
    boolean remove(Object o);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量查询操作。
     * 若列表包含c中所有元素则返回true，反之返回false。
     *
     * 本方法与前文介绍的contains(Object o)方法很相似，可对比学习。
     * 
     * @throws ClassCastException c中至少有一个元素的类型与列表不相容。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且列表不允许空元素存在。
     */
    boolean containsAll(Collection<?> c);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 将c中所有元素插入列表末尾，插入顺序为c的迭代器取出的顺序。
     * 若列表因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法未定义如下事件发生时的解决策略：在将c中的元素添加至列表的过程中c发生变化。
     * 这也意味着如下事件的解决策略同样未定义：将一个非空列表添加至自身。
     * 
     * 本方法与前文介绍的add(E e)方法很相似，可对比学习。
     * 本方法与后文介绍的方法addAll(int index, Collection<? extends E> c)很相似，可对比学习。
     * 本方法与后文介绍的方法add(int index, E element)很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws ClassCastException c中任意一个元素因其类型禁止被插入列表。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且列表不允许空元素存在。
     * @throws IllegalArgumentException c中任意一个元素因其某些属性禁止被插入列表。
     */
    boolean addAll(Collection<? extends E> c);

    /**
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 将c中所有元素插入列表的指定位置(index)，插入顺序为c的迭代器取出的顺序。
     * 原index位置及以后位置的元素顺次向后移动，空出装载c的空间。
     * 若列表因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法未定义如下事件发生时的解决策略：在将c中的元素添加至列表的过程中c发生变化。
     * 这也意味着如下事件的解决策略同样未定义：将一个非空列表添加至自身。
     * 
     * list.addAll(collection);
     * list.addAll(list.size(), collection);
     * 二者等价。
     * 
     * 本方法与前文介绍的add(E e)方法很相似，可对比学习。
     * 本方法与前文介绍的方法addAll(Collection<? extends E> c)很相似，可对比学习。
     * 本方法与后文介绍的方法add(int index, E element)很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws ClassCastException c中任意一个元素因其类型禁止被插入列表。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且列表不允许空元素存在。
     * @throws IllegalArgumentException c中任意一个元素因其某些属性禁止被插入列表。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    boolean addAll(int index, Collection<? extends E> c);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 从列表中移除所有与c的交集元素(求差集)。
     * 若列表因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法与前文介绍的remove(Object o)方法很相似，可对比学习。
     * 本方法与后文介绍的remove(int index)方法很相似，可对比学习。
     * 本方法与后文介绍的retainAll(Collection<?> c)方法正相反，可对比学习。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws ClassCastException c中至少有一个元素的类型与列表不相容。
     * @throws NullPointerException c==null或
     *                              列表中至少有一个元素为null且c不允许空元素存在。
     */
    boolean removeAll(Collection<?> c);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 保留列表中与c的交集元素。
     * 若列表因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法与前文介绍的removeAll(Collection<?> c)方法正相反，可对比学习。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws ClassCastException c中至少有一个元素的类型与列表不相容。
     * @throws NullPointerException c==null或
     *                              列表中至少有一个元素为null且c不允许空元素存在。
     */
    boolean retainAll(Collection<?> c);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 清空列表。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     */
    void clear();

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 比较及哈希操作。
     * 
     * 比较o与列表是否相等。当且仅当满足如下所有情况时返回true：
     * o为一个List。
     * o与list长度相同。
     * o与list所有对应位置上的元素相等(e1==null ? e2==null : e1.equals(e2))。
     * 
     * 换句话说，当两个列表在相同位置上的元素均相等，则认为二者相等。
     * 
     * 只要满足如上相等的定义，List接口的不同实现类也可被认定为相等。
     * 
     * 本方法与后文介绍的hashCode()方法成对出现，可对比学习。
     */
    boolean equals(Object o);

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 比较及哈希操作。
     * 
     * 返回列表的hash code值。其计算方式如下：
     * int hashCode = 1;
     * for (E e : list)
     *     hashCode = 31 * hashCode + (e == null ? 0 : e.hashCode());
     *
     * 这种计算方式可以保证任意list1.equals(list2)有list1.hashCode()==list2.hashCode()。
     * 
     * 本方法与前文介绍的equals(Object o)方法成对出现，可对比学习。
     */
    int hashCode();

    /**
     * 精确定位访问操作
     * 
     * 返回列表中索引位置为index的元素。
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    E get(int index);

    /**
     * 精确定位修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 用element替换列表索引值为index的元素。方法返回被替换的元素。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws ClassCastException element的类型与列表不相容。
     * @throws NullPointerException element==null且列表禁止包含空元素。
     * @throws IllegalArgumentException element因其某些属性禁止被插入列表。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    E set(int index, E element);

    /**
     * 精确定位修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 将element插入列表的index下标处。原来处于index下标及以后的元素均向后移动一个位置。
     * 
     * list.add("1");
     * list.add(list.size(), "1");
     * 等效
     * 
     * 本方法与前文介绍的add(E e)方法很相似，可对比学习。
     * 本方法与前文介绍的方法addAll(Collection<? extends E> c)很相似，可对比学习。
     * 本方法与前文介绍的方法addAll(int index, Collection<? extends E> c)很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws ClassCastException element的类型与列表不相容。
     * @throws NullPointerException element==null且列表禁止包含空元素。
     * @throws IllegalArgumentException element因其某些属性禁止被插入列表。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    void add(int index, E element);

    /**
     * 精确定位修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 移除列表中索引值为index的元素。后续元素左移一个位置。返回被移除的元素。
     * 
     * 本方法与前文介绍的remove(Object o)方法很相似，可对比学习。
     * 本方法与前文介绍的removeAll(Collection<?> c)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 列表不支持本方法。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    E remove(int index);

    /**
     * 搜索操作
     * 
     * 返回o在列表中第一次出现的索引。
     * 若列表不包含o则返回-1。
     * 更具体的说，返回满足如下条件索引值最小的元素的索引：
     * (o==null ? get(i)==null : o.equals(get(i)))。
     * 若不存在这样的元素则返回-1。
     * 
     * 本方法与后文介绍的lastIndexOf(Object o)方法正相反，可对比学习。
     * 
     * @throws ClassCastException o的类型与列表不相容。
     * @throws NullPointerException o==null且列表禁止包含空元素。
     */
    int indexOf(Object o);

    /**
     * 搜索操作
     * 
     * 返回o在列表中最后一次出现的索引。
     * 若列表不包含o则返回-1。
     * 更具体的说，返回满足如下条件索引值最大的元素的索引：
     * (o==null ? get(i)==null : o.equals(get(i)))。
     * 若不存在这样的元素则返回-1。
     * 
     * 本方法与前文介绍的indexOf(Object o)方法正相反，可对比学习。
     * 
     * @throws ClassCastException o的类型与列表不相容。
     * @throws NullPointerException o==null且列表禁止包含空元素。
     */
    int lastIndexOf(Object o);

    /**
     * 列表迭代器
     * 
     * 返回一个列表按固有顺序从头开始迭代的列表迭代器。
     * 
     * 本方法是前文介绍的iterator()方法基于List接口的特化，可对比学习。
     * 本方法与后文介绍的listIterator(int index)方法很相似，可对比学习。
     */
    ListIterator<E> listIterator();

    /**
     * 列表迭代器
     * 
     * 返回一个列表按固有顺序迭代的列表迭代器。
     * 迭代开始时游标位于索引[index-1,index]之间。
     * 即使用本方法得到列表迭代器后，若第一次调用的是listIterator.next()，返回的元素的索引是index；
     * 同理，若第一次调用的是listIterator.previous()，返回的元素的索引为index-1。
     * 
     * 本方法是前文介绍的iterator()方法基于List接口的特化，可对比学习。
     * 本方法与前文介绍的listIterator()方法很相似，可对比学习。
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    ListIterator<E> listIterator(int index);

    /**
     * 视图
     * 
     * 返回列表的一部分，索引值区间：[fromIndex,toIndex)。
     * 特别的，若fromIndex==toIndex，则返回空列表(不是null)。
     * 
     * 返回的列表是原列表的一个视图，前者是后者的一部分，并未从后者中分离出去。
     * 因此作用在前者之上的修改会反映在后者上，反之亦然。
     * 
     * 使用本方法的好处在于：
     * 当我们操作列表的一部分时，可以直接在这一部分内部操作(仿佛这就是一个新的列表)
     * 而无需被各种索引的边界分散过多的注意。
     * 
     * 例如，如果想批量移除列表中的一部分，可按如下操作：
     * list.subList(fromIndex, toIndex).clear();
     * 
     * 同理，所有本接口的方法及所有java.util.Collections类提供的静态方法
     * 均支持本方法返回的视图作为一个独立的列表调用。例如：
     * list2 = list.subList(1, 2);
     * list2.get(0);
     * 此时取得的元素即为list.get(1)。
     * 
     * 若在生成视图后原列表发生了结构性的变化(例如长度发生了变化)
     * 则已生成的视图将被重置为未定义的状态。
     * 
     * @throws IndexOutOfBoundsException 索引越界(fromIndex < 0 || toIndex > size || fromIndex > toIndex)。
     */
    List<E> subList(int fromIndex, int toIndex);
}
```

# 已整理层级关系

***直接父接口***

- [java.util.Collection&lt;E&gt;](/2017/05/23/Java JDK7源码-javautilCollectionE/)

***直接实现本接口的类***

- [java.util.AbstractList&lt;E&gt;](/2017/06/19/Java JDK7源码-javautilAbstractListE/)
- [java.util.ArrayList&lt;E&gt;](/2017/07/06/Java JDK7源码-javautilArrayListE/)

# 综述

本接口是Java集合框架中的一员。List(列表)又名Sequence(序列)。是一种有序的集合。

使用者能精确的知道列表中每个元素的位置，能通过元素在列表中的索引访问或搜索元素。和数组一样，列表的索引从0开始。

---

不同于Set，列表允许重复元素。更通俗的说，若设e1,e2为列表中的两个元素，则允许e1.equals(e2)。特别的，若列表中允许空元素，则其也允许复数个空元素。

本接口不支持如下这种列表：禁止重复，且在试图插入重复元素时抛出运行时异常。

---

较之父接口Collection，本接口在如下方法中规定了额外的规范：

- iterator()
- add(E e)
- remove(Object o)
- equals(Object o)
- hashCode()

为方便起见，父接口Collection中的其他方法本接口也会再写一遍。

---

较之父接口Collection，本接口增加了4个可基于索引访问元素的方法:

- get(int index)
- set(int index, E element)
- add(int index, E element)
- remove(int index)

对本接口的某些实现而言(例如LinkedList)，上述操作的时间复杂度与索引值正相关(链表为取到某个索引值的元素需在链式结构中逐个向后查找)。因此在不确定本接口的具体实现时，比起使用上述基于索引的访问元素的方法，更推荐使用迭代器。

---

较之父接口Collection，本接口新增了一个特殊的迭代器ListIterator，它是Iterator的子接口。

较之父接口Iterator，ListIterator增加了新的功能：

- 元素插入
- 元素替换
- 双向访问

较之父接口Collection，本接口新增了一个方法新建从头开始迭代的ListIterator，同时还新增了另一个新建从特定索引开始的ListIterator的方法。

---

较之父接口Collection，本接口新增了两个方法用于搜索特定的对象：

- indexOf(Object o)
- lastIndexOf(Object o)

从性能的角度考虑，这两个方法应谨慎使用。在很多实现中，这两个方法采用代价昂贵的顺序查找(linear search)。

---

较之父接口Collection，本接口提供了两个方法用于在任意位置高效的插入或删除复数个元素：

- addAll(int index, Collection&lt;? extends E&gt; c)
- removeAll(Collection&lt;?&gt; c)

---

当列表允许包含自身以作为自身的元素时，这种列表中的equals(Object o)方法及hashCode()方法往往难于定义。

---

本接口的某些实现会对它们所能包含的元素有所限制。

例如，一些实现禁止包含空元素，一些实现对其所能包含的元素的类型有限制。

试图插入一个不合规的元素会抛出一个运行时异常，例如NullPointerException，ClassCastException。

依具体实现不同试图查询一个不合规的元素可能会抛出一个异常，或者仅仅只是返回false。

# 未整理层级关系

***直接子接口***

- [com.sun.org.apache.xerces.internal.xs.datatypes.ObjectList]()
- [com.sun.org.apache.xerces.internal.xs.datatypes.ByteList]()
- [com.sun.org.apache.xerces.internal.xs.ShortList]()
- [com.sun.org.apache.xerces.internal.xs.StringList]()
- [com.sun.org.apache.xerces.internal.xs.XSObjectList]()
- [com.sun.org.apache.xerces.internal.xs.XSNamespaceItemList]()
- [com.sun.corba.se.spi.ior.IOR]()
- [com.sun.corba.se.spi.ior.IORTemplate]()
- [com.sun.corba.se.spi.ior.IORTemplateList]()
- [com.sun.corba.se.spi.ior.TaggedProfileTemplate]()
- [org.w3c.dom.ls.LSInput.LSInputList]()

***直接实现本接口的类***

- [java.util.LinkedList&lt;E&gt;]()
- [java.util.Vector&lt;E&gt;]()
- [java.util.concurrent.CopyOnWriteArrayList&lt;E&gt;]()

***直接实现本接口的内部类***

- [静态成员内部类java.util.Collections.CheckedList&lt;E&gt;]()
- [静态成员内部类java.util.Collections.SynchronizedList&lt;E&gt;]()
- [静态成员内部类java.util.Collections.UnmodifiableList&lt;E&gt;]()