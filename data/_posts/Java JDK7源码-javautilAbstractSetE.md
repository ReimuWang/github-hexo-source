---
title: Java JDK7源码-java.util.AbstractSet&lt;E&gt;
date: 2017-06-28 11:12:36
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {

    protected AbstractSet() {
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 实现接口:java.util.Set<E>
     * 
     * 比较o与set的相等性。
     * 满足如下条件返回true：
     * o同样是一个Set
     * o与set长度相同
     * o中的每个元素都在set中存在
     * 只要满足这些条件，即便是Set接口的不同实现之间也可判定为相等
     * 
     * 本方法首先检查o是否是set本身，如果是则返回true。
     * 然后检查o是否是一个Set及长度是否与set相同，如果有一个为否则返回false。
     * 如果均为true，则返回containsAll((Collection) o)的结果。
     */
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Set))
            return false;
        Collection c = (Collection) o;
        if (c.size() != size())
            return false;
        try {
            return containsAll(c);
        } catch (ClassCastException unused)   {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 实现接口:java.util.Set<E>
     * 
     * 返回set的hash code值。
     * 
     * 一个set的hash code值为其组成元素的hash code值之和
     * 等于null的元素的hash code值被定义为0
     * 这样可以保证对于任意两个set，s1及s2
     * 只要有s1.equals(s2)，即有s1.hashCode()==s2.hashCode()
     * 满足Java规范对于hashCode()及equals(Object obj)返回关系的要求
     * 
     * 本方法迭代set
     * 调用set中每个元素的hashCode()
     * 求和作为本方法的结果返回
     */
    public int hashCode() {
        int h = 0;
        Iterator<E> i = iterator();
        while (i.hasNext()) {
            E obj = i.next();
            if (obj != null)
                h += obj.hashCode();
        }
        return h;
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 实现接口:java.util.Set<E>
     * 
     * 本方法可选
     * 移除set中所有被c所包含的元素
     * 若c同样是一个Set，本操作实际为求差集
     * 
     * 本方法通过调用set及c的size()确定哪一个的长度更小:
     * 
     * 若set长度更小，则迭代set
     * 检查迭代出的元素是否被c所包含
     * 若包含，则调用迭代器的remove()将其从set中移除。
     * 
     * 若c的长度更小，则迭代c
     * 调用set的remove(Object o)从set中移除所有被迭代器返回的元素。
     * 
     * 注意，若set返回的iterator未实现remove()
     * 则会抛出UnsupportedOperationException。
     * 
     * 若set因本方法发生了改变则返回true，反之返回false。
     * 
     * 本方法所重写的AbstractCollection类下的方法为：
     * public boolean removeAll(Collection<?> c) {
     *     boolean modified = false;
     *     Iterator<?> it = iterator();
     *     while (it.hasNext()) {
     *         if (c.contains(it.next())) {
     *             it.remove();
     *             modified = true;
     *         }
     *     }
     *     return modified;
     * }
     * 因为Set不允许包含重复元素
     * 本方法重写后增加了基于长度的判断以提高算法性能。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException 集合中至少有一个元素的类型与c不相容。
     * @throws NullPointerException c==null或
     *                              集合中至少有一个元素为null且c不允许空元素存在。
     */
    public boolean removeAll(Collection<?> c) {
        boolean modified = false;

        if (size() > c.size()) {
            for (Iterator<?> i = c.iterator(); i.hasNext(); )
                // 因Set不允许包含重复元素
                // 故调用1次remove后即可保证set中不再包含被移除的元素
                modified |= remove(i.next());
        } else {
            for (Iterator<?> i = iterator(); i.hasNext(); ) {
                if (c.contains(i.next())) {
                    i.remove();
                    modified = true;
                }
            }
        }
        return modified;
    }

}
```

# 已整理层级关系

***本类直接继承的类***

- [java.util.AbstractCollection&lt;E&gt;](/2017/06/19/Java JDK7源码-javautilAbstractCollectionE/)

***本类直接实现的接口***

- [java.util.Set&lt;E&gt;](/2017/06/13/Java JDK7源码-javautilSetE/)

# 综述

本类是Java集合框架中的一员。提供了Set接口的最小化实现。E表示被Set所持有的元素类型。

在遵循Set接口的额外约束(例如不允许存在重复元素)的前提下，继承本类实现Set与继承AbstractCollection实现Collection所需做的工作是相同的。

# 未整理层级关系

***直接继承本类的类***

- [java.util.HashSet<E>]()