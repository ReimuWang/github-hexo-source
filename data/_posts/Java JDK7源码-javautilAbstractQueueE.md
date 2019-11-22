---
title: Java JDK7源码-java.util.AbstractQueue&lt;E&gt;
date: 2017-06-30 11:23:14
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public abstract class AbstractQueue<E>
    extends AbstractCollection<E>
    implements Queue<E> {

    protected AbstractQueue() {
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 实现接口:java.util.Queue<E>
     * 
     * 在不超出容量限制的前提下
     * 将e插入到queue中并返回true
     * 若已超出容量限制无法插入
     * 则抛出IllegalStateException
     * 
     * 本方法通过调用offer(E e)实现。
     * 若offer(E e)返回true则本方法返回true
     * 若offer(E e)返回false则本方法抛出IllegalStateException。
     * 
     * @throws IllegalStateException 因容量限制e此时无法被插入queue。
     * @throws ClassCastException e因其所属的类禁止被插入queue。
     * @throws NullPointerException e==null且queue禁止包含空元素。
     * @throws IllegalArgumentException e因其某些属性禁止被插入queue。
     */
    public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }

    /**
     * 实现接口:java.util.Queue<E>
     * 
     * 检索，移除并返回queue的头元素。
     * 
     * 本方法与poll()的区别仅仅在于：
     * 若queue为空队列，poll()返回null，本方法抛出异常。
     * 因此，除非queue为空，否则本方法返回poll()的结果。
     * 
     * @throws NoSuchElementException queue为空。
     */
    public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

    /**
     * 实现接口:java.util.Queue<E>
     * 
     * 检索并返回但不移除queue的头元素。
     * 
     * 本方法与peek()的区别仅仅在于：
     * 若queue为空队列，peek()返回null，本方法抛出异常。
     * 因此，除非queue为空，否则本方法返回peek()的结果。
     * 
     * @throws NoSuchElementException queue为空。
     */
    public E element() {
        E x = peek();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 
     * 移除queue中的所有元素。
     * 调用本方法后，queue将为空队列。
     * 
     * 本方法重复调用poll()直到其返回null为止。
     */
    public void clear() {
        while (poll() != null)
            ;
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 
     * 将c中所有元素加入queue。
     * 若c为queue自身则抛出IllegalArgumentException。
     * 更一般的说，本方法并未定义如下行为的解决策略：c在本方法操作过程中发生变化。
     * 
     * 本方法迭代c同时将迭代得到的每个元素插入queue。
     * 
     * 在插入某个元素(特别的，这个某个元素也包含null)时若因相关异常导致插入失败
     * 结果只有部分元素能成功插入queue，此时会抛出运行时异常。
     * 
     * queue因本方法发生改变则返回true。
     * 
     * @throws ClassCastException c中任意一个元素因为其所属的类禁止被插入queue。
     * @throws NullPointerException c中任意一个元素为null且queue禁止包含空元素；或c==null。
     * @throws IllegalArgumentException c中任意一个元素因其某些属性禁止被插入queue；或c是queue自身。
     * @throws IllegalStateException 因插入限制，此时并非c中所有元素都能被插入queue。
     */
    public boolean addAll(Collection<? extends E> c) {
        if (c == null)
            throw new NullPointerException();
        if (c == this)
            throw new IllegalArgumentException();
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

}
```

# 已整理层级关系

***本类直接继承的类***

- [java.util.AbstractCollection&lt;E&gt;](/2017/06/19/Java JDK7源码-javautilAbstractCollectionE/)

***本类直接实现的接口***

- [java.util.Queue&lt;E&gt;](/2017/06/16/Java JDK7源码-javautilQueueE/)

# 综述

本类是Java集合框架中的一员。E为AbstractQueue所包含的元素类型。

本类提供Queue接口的基本实现。若基本实现要求不得包含空元素，本类方法会对此作出调整。add(E e)，remove()，element()分别基于offer(E e)，poll()，peek()。不同的是当操作失败时前者抛出异常，后者会返回失败标志(false或null)。

若要以最小化的方式实现本类，程序员必须实现Queue.offer(E e)(禁止插入空元素)，Queue.peek()，Queue.poll()，Collection.size()，Collection.iterator()。若实现类无法满足Queue的规范，则应考虑使用AbstractCollection或其其他子类。

# 未整理层级关系

***直接继承本类的类***

- [java.util.concurrent.ArrayBlockingQueue&lt;E&gt;]()
- [java.util.concurrent.LinkedBlockingDeque&lt;E&gt;]()
- [java.util.concurrent.ConcurrentLinkedQueue&lt;E&gt;]()
- [java.util.concurrent.DelayQueue&lt;E extends Delayed&gt;]()
- [java.util.PriorityQueue&lt;E&gt;]()