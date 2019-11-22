---
title: Java JDK7源码-java.util.Queue&lt;E&gt;
date: 2017-06-16 11:49:21
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public interface Queue<E> extends Collection<E> {

    /**
     * 继承父接口:java.util.Collection<E>
     * 
     * 在不超过队列容量限制的情况下，将e插入到队列中(具体的插入位置依队列的优先规则而定)。
     * 若插入成功则返回true。若因容量不足而插入失败则抛出IllegalStateException。
     * 
     * 本方法与后文介绍的offer(E e)方法很相似，可对比学习。
     * 
     * @throws IllegalStateException 因容量限制e此时无法被插入队列。
     * @throws ClassCastException e因其类型禁止被插入队列。
     * @throws NullPointerException e==null且队列禁止包含null。
     * @throws IllegalArgumentException e因其某些属性禁止被插入队列。
     */
    boolean add(E e);

    /**
     * 在不超过队列容量限制的情况下，将e插入到队列中(具体的插入位置依队列的优先规则而定)。
     * 若插入成功则返回true。若因容量不足而插入失败则返回false。
     * 
     * 当队列有容量限制时，本方法通常优于add(E e)。
     * 因为对于有容量限制的队列而言，因容量已满无法插入元素是正常情况，不需要抛出异常。
     * 
     * 本方法与前文介绍的add(E e)方法很相似，可对比学习。
     * 
     * @throws ClassCastException e因其类型禁止被插入队列。
     * @throws NullPointerException e==null且队列禁止包含null。
     * @throws IllegalArgumentException e因其某些属性禁止被插入队列。
     */
    boolean offer(E e);

    /**
     * 本方法与父接口Collection中的remove(Object o)方法是重载关系，不是重写关系，注意不要混淆。
     * 
     * 返回并移除队列的头元素(头元素依队列的优先规则计算而得)。
     * 
     * 本方法与poll()方法的差异只体现在队列为空时：本方法抛出异常，poll()方法返回null。
     * 
     * 本方法与后文介绍的poll()方法很相似，可对比学习。
     * 
     * @throws NoSuchElementException 队列为空
     */
    E remove();

    /**
     * 返回并移除队列的头元素(头元素依队列的优先规则计算而得)。
     * 
     * 本方法与remove()方法的差异只体现在队列为空时：本方法返回null，remove()方法抛出异常。
     * 
     * 本方法与前文介绍的remove()方法很相似，可对比学习。
     */
    E poll();

    /**
     * 返回但不移除队列的头元素(头元素依队列的优先规则计算而得)。
     * 
     * 本方法与peek()方法的差异只体现在队列为空时：本方法抛出异常，peek()方法返回null。
     * 
     * 本方法与后文介绍的peek()方法很相似，可对比学习。
     * 
     * @throws NoSuchElementException 队列为空
     */
    E element();

    /**
     * 返回但不移除队列的头元素(头元素依队列的优先规则计算而得)。
     * 
     * 本方法与element()方法的差异只体现在队列为空时：本方法返回null，element()方法抛出异常。
     * 
     * 本方法与前文介绍的element()方法很相似，可对比学习。
     */
    E peek();
}
```

# 已整理层级关系

***直接父接口***

- [java.util.Collection&lt;E&gt;](/2017/05/23/Java JDK7源码-javautilCollectionE/)

***直接实现本接口的类***

- [java.util.AbstractQueue&lt;E&gt;](/2017/06/30/Java JDK7源码-javautilAbstractQueueE/)

# 综述

本接口是Java集合框架中的一员。Queue&lt;E&gt;中的E表示Queue中元素的类型。

---

在数据结构中，队列是一种按照设定好的优先规则进行排序的集合，移除时只能移除头部元素。这是广义上队列的定义。

狭义的队列，也是我们通常所说的队列，优先规则为入队的时间先后：入队越早，优先级越高。此时上文中的广义定义退化为FIFO(first-in-first-out)。

而广义的队列中的优先规则则可以是任意规则，比如我们甚至可以将狭义的优先规则颠倒过来：优先规则的依据依然是时间，只不过入队越晚，优先级越高。此时队列实现的就是LIFO(last-in-first-out)，也就是栈的功能了。所以从广义的角度上讲，栈其实是广义队列的一种。

Java中的Queue接口约束的是广义的队列。不过除非特殊声明，我们通常提到Java中的Queue接口时，指得依然是狭义上的队列，并且除非真的有必要，也没必要去实现广义上的队列，徒增程序的复杂性。

正因为Queue接口将优先规则交由实现类自行规定，因此实现类应在其文档中将自身采用的优先规则写明，以免其使用者产生误解。

---

与Collection接口相比，Queue接口提供了额外的插入，取出，校验操作。在操作失败时每个方法的返回值可能为以下两种返回形式之一：

- 抛出一个异常
- 返回一个表示失败的值(依操作不同为null或false)

![0.jpg](/images/blog_pic/Java JDK7源码/javautilQueueE/0.jpg)

其中抛出异常的那一组方法继承自父接口Collection(或沿用Collection接口的思路)，而返回特殊值的那一组是Queue接口额外添加的。

这是由Java的异常机制所致的。以添加元素为例，对于普通的集合而言，集合不足以容纳新元素是一种异常情况。而对于队列而言，因队列已满无法插入却属于正常情况(例如元素因无法插入固定容量的队列而需阻塞时)。为了解决这个矛盾，只能在遵循父接口的插入规范的基础上(毕竟抛出异常才是通常情况，不能因为队列这个特例改变Collection这一级别的设定)，重新定义符合自身逻辑规范的方法。

上图中还有一个需要注意的点就是poll()方法及peek()方法在队列为空时返回的都是null。这就意味着若这两个方法返回了null，我们是无法确定到底是队列为空了还是返回的元素就是null。因此虽然Queue接口未做强制规范，其实现类往往也会禁止队列包含空元素。

这6个方法就是全部显式声明在Queue接口中的方法了，其他方法均默认继承父接口Collection。这也从另一个侧面说明了这6个方法之于Queue接口的特殊性。

---

Java中的队列大多被用于并发环境下的阻塞队列。不过Queue接口中并未做并发规范，该规范均被定义在了Queue接口的直接子接口BlockingQueue中。

---

在equals(Object o)方法及hashCode()方法的设计上，Queue接口并未强制要求其实现类必须重写(因为优先规则实际上就是一种比较，没必要再做特别规范引起程序员的注意了)，默认情况下将调用公共父类Object中的equals(Object o)方法及hashCode()方法。

# 未整理层级关系

***直接子接口***

- [java.util.Deque&lt;E&gt;]()
- [java.util.concurrent.BlockingQueue&lt;E&gt;]()
- [java.util.concurrent.ConcurrentLinkedQueue&lt;E&gt;]()

***直接实现本接口的内部类***

- [静态成员内部类java.util.Collections.AsLIFOQueue&lt;E&gt;]()