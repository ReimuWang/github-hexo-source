---
title: Java JDK7源码-java.util.Collection&lt;E&gt;
date: 2017-05-23 23:37:07
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public interface Collection<E> extends Iterable<E> {

    /**
     * 查询操作。
     * 返回集合包含的元素个数。
     * 若集合包含的元素个数大于Integer.MAX_VALUE，则返回Integer.MAX_VALUE。
     */
    int size();

    /**
     * 查询操作。
     * 若集合不包含元素则返回true。反之返回false。
     */
    boolean isEmpty();

    /**
     * 查询操作。
     * 若集合包含o则返回true，反之返回false。
     * 更具体的说，当且仅当集合包含至少一个元素e，满足(o==null ? e==null : o.equals(e))时，返回true，反之返回false。
     * 
     * 本方法与后文介绍的containsAll(Collection<?> c)方法很相似，可对比学习。
     * 
     * @throws ClassCastException o的类型与集合不相容。
     * @throws NullPointerException o==null且集合不允许空元素存在。
     */
    boolean contains(Object o);

    /**
     * 继承父接口:java.lang.Iterable<T>
     * 
     * 查询操作。
     * 返回一个集合的迭代器，该迭代器默认从头开始迭代。
     * 本接口不保证集合本身是否有序，即不保证迭代器是否有序，均交由子孙自行控制。
     */
    Iterator<E> iterator();

    /**
     * 查询操作。
     * 本方法是连接集合与数组之间的桥梁。
     * 返回包含集合所有元素的数组。若集合是有序的(即迭代器返回的结果有序)，则返回的数组必须有同样的顺序。
     * 返回的数组与集合之间不存在引用关系(即使集合的底层就是基于数组实现的)，数组中的元素是集合中元素的浅拷贝。
     * 
     * 本方法与后文介绍的toArray(T[] a)方法很相似，可对比学习。
     */
    Object[] toArray();

    /**
     * 查询操作。
     * 本方法是连接集合与数组之间的桥梁。
     * 
     * 返回包含集合所有元素的数组。返回数组的类型即为a的类型。若集合是有序的(即迭代器返回的结果有序)，则返回的数组必须有同样的顺序。
     * 具体规则的伪代码为：
     * if (a.length < collection.size())
     *     不修改a，而是以a的类型新建长度为collection.size()的数组并填入集合中的值，随后返回这个新生成的数组。
     * else if (a.length == collection.size()
     *     将返回的结果直接填入a后返回a(若a中已有值，则a中的原值会被覆盖)。
     * else
     *     数组索引在[0, collection.size() -1]的元素会被集合对应位置的元素覆盖。
     *     数组索引 == collection.size()的元素会被置为null
     *     数组中后续元素(如果有的话)不变。
     * 
     * 若集合不允许包含空元素，则本方法此时可用来计算集合的长度：返回的数组第一次出现null的索引即为集合的长度。
     * 
     * 返回的数组与集合之间不存在引用关系(即使集合的底层就是基于数组实现的)，数组中的元素是集合中元素的浅拷贝。
     * 
     * 小例子：
     * String[] y = x.toArray(new String[0]);
     * x是一个元素类型为String的集合，则上述语句会将x中的元素依序复制一份浅拷贝到数组y。
     * 
     * 关于本方法与前文介绍的toArray()方法，需明确：
     * collection.toArray()
     * collection.toArray(new Object[s])
     * 当s < collection.size()时，上述两行代码的效果等价。
     * 
     * 本方法与前文介绍的toArray()方法很相似，可对比学习。
     * 
     * @throws ArrayStoreException a的类型collection不支持。
     * @throws NullPointerException null==a。
     */
    <T> T[] toArray(T[] a);

    /**
     * 修改操作。
     * 本方法属于破坏性方法，可选。
     * 
     * 若实现类不支持本方法，则应抛出UnsupportedOperationException。本机制尤其用于保护那些元素为常量值的集合。
     * 
     * 若集合因本方法而改变则返回true，反之则返回false。
     * 返回false的情况举例：集合不允许包含重复元素且e在集合中已存在。
     * 因此对于那些不能包含重复元素的集合而言，该方法也可用于判断集合是否已包含e(不过需要注意，如果不包含e会被插入集合)。
     * 
     * 即便实现类支持本方法，它也可能会做某种限制(禁止插入null，限制可插入的元素类型等)。
     * 
     * 本方法与后文介绍的addAll(Collection<? extends E> c)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException e因其类型禁止被插入集合。
     * @throws NullPointerException e==null且集合禁止包含null。
     * @throws IllegalArgumentException e因其某些属性禁止被插入集合。
     * @throws IllegalStateException 因插入限制，此时e不能被插入集合。
     */
    boolean add(E e);

    /**
     * 修改操作。
     * 本方法属于破坏性方法，可选。
     * 
     * 移除集合符合条件的元素e：
     * (o==null ? e==null : o.equals(e))
     * 若有多个e符合条件，则本方法只会移除其中一个，具体移除哪一个依具体实现而定。
     * 
     * 若本方法改变了集合则返回true，反之返回false。
     * 
     * 本方法与后文介绍的removeAll(Collection<?> c)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException o的类型与集合不相容。
     * @throws NullPointerException o==null且集合不允许空元素存在。
     */
    boolean remove(Object o);

    /**
     * 批量查询操作。
     * 若集合包含c中所有元素则返回true，反之返回false。
     *
     * 本方法与前文介绍的contains(Object o)方法很相似，可对比学习。
     * 
     * @throws ClassCastException c中至少有一个元素的类型与集合不相容。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且集合不允许空元素存在。
     */
    boolean containsAll(Collection<?> c);

    /**
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 将c中所有元素插入集合。
     * 若集合因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法未定义如下事件发生时的解决策略：在将c中的元素添加至集合的过程中c发生变化。
     * 这也意味着如下事件的解决策略同样未定义：将一个非空集合添加至自身。
     * 
     * 本方法与前文介绍的add(E e)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException c中任意一个元素因其类型禁止被插入集合。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且集合不允许空元素存在。
     * @throws IllegalArgumentException c中任意一个元素因其某些属性禁止被插入集合。
     * @throws IllegalStateException 因插入限制，此时并非c中所有元素都能被插入集合。
     */
    boolean addAll(Collection<? extends E> c);

    /**
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 从集合中移除所有与c的交集元素(求差集)。
     * 若集合因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法与前文介绍的remove(Object o)方法很相似，可对比学习。
     * 本方法与后文介绍的retainAll(Collection<?> c)方法正相反，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException c中至少有一个元素的类型与集合不相容。
     * @throws NullPointerException c==null或
     *                              集合中至少有一个元素为null且c不允许空元素存在。
     */
    boolean removeAll(Collection<?> c);

    /**
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 保留集合中与c的交集元素。
     * 若集合因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法与前文介绍的removeAll(Collection<?> c)方法正相反，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException c中至少有一个元素的类型与集合不相容。
     * @throws NullPointerException c==null或
     *                              集合中至少有一个元素为null且c不允许空元素存在。
     */
    boolean retainAll(Collection<?> c);

    /**
     * 批量修改操作
     * 本方法属于破坏性方法，可选。
     * 
     * 清空集合。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     */
    void clear();

    /**
     * 比较及哈希操作。
     * 
     * 比较o与集合是否相等。若相等则返回true，反之返回false。
     * 关于判等的逻辑，若实现类未显式重写本方法，本方法将默认调用公共父类Object类的equals(Object obj)方法，即基于集合的引用地址判断是否相等。
     * 
     * 事实上，因为实现类必然会是公共父类Object类的子类，因此纯从规范的角度来说，本接口没必要显式声明本方法。
     * 之所以还是要显式声明，是为了强调本方法的重要性，提醒程序员在写本接口的实现类时重视并重写本方法。
     * 后文介绍的hashCode()方法也是同理。
     * 
     * 一般来说，设计本方法时应遵循如下规范：
     * 自反性，即a.equals(a)应为true。
     * 对称性，即若a.equals(b)为true，则应有b.equals(a)为true。
     * 传递性，即若a.equals(b)为true，b.equals(c)为true，则应有a.equals(c)为true。
     * 
     * Java API提供的直接继承本接口的子接口有3个：Queue,List,Set。
     * 其中List,Set均对本方法有要求，它们要求其实现类的本方法不是继承自公共父类Object类，而是根据其具体的特点，比较集合中元素的"值"是否相等。
     * 因此，当程序员使用Queue接口，或者是自己直接实现本接口时，一定要考虑好本方法的设计。
     * 因为通常情况下，我们比较两个集合的相等性时，都不希望比较的是集合的引用地址是否相等，因为这往往是毫无意义的。
     * 
     * Java集合框架在设计时规定了一个大前提：List只能与List相等，Set只能与Set相等。
     * 因此，我们无法写出一个类同时实现Set及List接口：若这样的实现类可以存在，意味着有一种集合既是List又是Set，与上文介绍的大前提相悖。
     * 
     * 本方法与后文介绍的hashCode()方法成对出现，可对比学习。
     */
    boolean equals(Object o);

    /**
     * 比较及哈希操作。
     * 
     * 返回集合的hash code值。
     * 若实现类未显式重写本方法，本方法将默认调用公共父类Object类的hashCode()方法。
     * 
     * 本方法与equals(Object o)方法成对出现，可对比学习。
     * 换句话说，只要重写了equals(Object o)方法，就必须配对重写本方法。
     * 具体来说，若c1.equals(c2)为true，则应有c1.hashCode()==c2.hashCode()。
     */
    int hashCode();
}
```

# 已整理层级关系

***直接父接口***

- [java.lang.Iterable&lt;T&gt;](/2017/05/23/Java JDK7源码-javalangIterableT/)

***直接子接口***

- [java.util.List&lt;E&gt;](/2017/05/25/Java JDK7源码-javautilListE/)
- [java.util.Set&lt;E&gt;](/2017/06/13/Java JDK7源码-javautilSetE/)
- [java.util.Queue&lt;E&gt;](/2017/06/16/Java JDK7源码-javautilQueueE/)

***直接实现本接口的类***

- [java.util.AbstractCollection&lt;E&gt;](/2017/06/19/Java JDK7源码-javautilAbstractCollectionE/)

# 综述

本接口是Java集合框架中的一员。或者更具体的说，是Java集合框架层级中的根接口。Collection&lt;E&gt;中的E代表集合中元素的类型。

---

Java API不会提供本接口的直接实现类，而是会实现本接口的子接口(Set,List,Queue)。通常情况下，这些实现类都应提供两个"标准"的构造函数(当然，因为接口无法包含构造函数，因此这个规范并不具备强制力，不过Java API中所有实现了本接口的类都遵循这个规范，Java语言规范中也强烈推荐程序员自定义的本接口的实现类也遵循这一规范)：

- 无参构造函数，它会创建一个空的集合。
- 带有一个集合类型参数的构造函数，它会依传入参数创建一个新的集合(新集合可与传入参数的类型不同，例如可以传入List，最终构造出Set)。

---

本接口包含破坏性方法(即可能会改变所属集合的方法)。若某个具体的实现类不需要支持某个破坏性方法，则需要抛出UnsupportedOperationException。

这样做的目的是为了引起调用者的重视：改变集合属于危险操作(相对于只读方法而言)，但是当前实现类不支持这个方法，你是不是对本实现类有什么误解呢？

不过需要注意的是，目的才是最重要的，而抛出UnsupportedOperationException只是手段。换句话说，这一约束即便在官方的推荐中也没有那么绝对。有时，即便某实现类不支持某破坏性方法，但只要该方法不产生真正的破坏性结果，那么也可以不抛出UnsupportedOperationException。例如：某不可变实现类不支持addAll(Collection&lt;? extends E&gt; c)方法，在null==c时，就可以选择不抛出UnsupportedOperationException。

---

某些本接口的实现类可能会对其所包含的元素有更具体的限制。例如，有些实现禁止插入空元素，有些实现则会限制元素类型。试图插入不合法元素时官方推荐抛出一个unchecked exception，通常为NullPointerException或ClassCastException。试图查询一个不合法元素时官方推荐抛出异常或返回false。

---

本接口不规定同步策略，交由子孙自行控制。

---

本接口中的方法在判等时多使用元素的equals()方法。因为这样比较出的才是元素在业务逻辑层面的相等性。以contains(Object o)为例：

```
当且仅当该集合包含至少一个元素e，满足(o==null ? e==null : o.equals(e))时，返回true。
```

当然，这个描述只是一个整体上的指导思路。并不是说具体实现类在实现这个方法时，就一定要遍历集合，或者一定要调用元素的equals()方法。事实上，只要能实现功能，实现类可以灵活的进行优化。例如某些情况下，实现类也可用hashCode()方法替代equals()方法比较两元素的相等性。

# 未整理层级关系

***直接子接口***

- [java.beans.beancontext.BeanContext]()

***直接实现本接口的内部类***

- [静态成员内部类java.util.Collections.CheckedCollection&lt;E&gt;]()
- [静态成员内部类java.util.Collections.SynchronizedCollection&lt;E&gt;]()
- [静态成员内部类java.util.Collections.UnmodifiableCollection&lt;E&gt;]()