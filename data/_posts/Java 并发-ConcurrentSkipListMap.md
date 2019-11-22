---
title: Java 并发-ConcurrentSkipListMap
date: 2018-02-07 15:38:36
tags: [Java,并发]
categories: Java 并发
---

# 跳表

跳表(SkipList)是一种可用来进行快速查找的数据结构，时间复杂度为O(logn)，有点类似于平衡树。之所以这么说，是因为它们都可以对元素进行快速的查找。但二者有一个重要的区别：对平衡树的修改(也就是所谓的插入和删除)往往很可能会导致平衡树进行一次全局的调整(也就是所谓的调平)，而对跳表而言，插入和删除只需要对整个数据结构的局部进行调整即可。这样的好处是显而易见的：

1. 需要调整的规模较小，且数据量越大所带来的性能提升自然也越大。

2. 在并发环境下，当需要修改跳表时，我们不需要将整张表锁起来，而可以只锁住本次修改所影响的区域，提高性能。

<!-- more -->

跳表的另一个特点就是随机算法。跳表的本质是同时维护多个链表，且这些链表是分层的。如下图所示：

![0.jpg](/images/blog_pic/Java 并发/ConcurrentSkipListMap/0.jpg)

最底层的链表包含跳表的所有元素，每往上一层的链表都是下面链表的子集(又是典型的用空间换时间的策略)，一个元素被插入哪些层完全是随机的(当然，最下面那层是必然会被插入的)，因此如果运气不好的话，也许会得到一个性能很糟糕的结构。不过在实际使用中，尤其是数据量较大时，概率将会极大的掩盖运气，跳表的性能通常都是非常好的。

跳表内所有链表的元素都是具有偏序关系的(所以才说跳表和平衡树像嘛)。查找时，通常是从顶层链表开始找起，一旦发现元素值大于待搜索值，或者找到末尾也未找到，则说明本层已无找到的可能，便进入下一层，直到找到或是遍历完底层也没找到(此时说明确实没有)。

下图是一个查找值7的小例子：

![1.jpg](/images/blog_pic/Java 并发/ConcurrentSkipListMap/1.jpg)

# ConcurrentSkipListMap

java API中提供的跳表为java.util.concurrent.ConcurrentSkipListMap，类定义如下：

```
public class ConcurrentSkipListMap<K,V> extends AbstractMap<K,V> implements ConcurrentNavigableMap<K,V>, Cloneable, java.io.Serializable
```

这是一个Map，它与我们在Java API中最常见的使用哈希算法实现的Map有一个显著的不同：哈希并不会保存元素的顺序，而跳表因其特性其元素自然是有序的(所以说，并非所有Map都是无序的)。因此如果需要一个有序的Map，那么跳表可能是很好的选择。

我们可以先来看一个小例子：

```
import java.util.Map;
import java.util.concurrent.ConcurrentSkipListMap;

public class Test {

    public static void main(String[] args) {
        Map<Integer, Integer> map = new ConcurrentSkipListMap<>();
        for (int i = 0; i < 5; i++) map.put(i, i);
        for (Map.Entry<Integer, Integer> entry : map.entrySet())
            System.out.println(entry.getKey());
    }
}
```

输出：

```
0
1
2
3
4
```

既然跳表底层是以链表实现的，那么它最重要的数据结构自然就是节点了。对于ConcurrentSkipListMap而言，节点类是他的静态成员内部类。全部代码如下：

```
static final class Node<K,V> {
    // key与value组合构成了链表节点本身存储的元素值
    final K key;    // 即map的key
    volatile Object value;    // 即map的value
    volatile Node<K,V> next;    // 下一个节点

    Node(K key, Object value, Node<K,V> next) {
        this.key = key;
        this.value = value;
        this.next = next;
    }

    Node(Node<K,V> next) {
        this.key = null;
        this.value = this;
        this.next = next;
    }

    /**
     * 设置value字段
     * 使用CAS来进行并发控制
     */
    boolean casValue(Object cmp, Object val) {
        return UNSAFE.compareAndSwapObject(this, valueOffset, cmp, val);
    }

    /**
     * 设置next字段
     * 使用CAS来进行并发控制
     */
    boolean casNext(Node<K,V> cmp, Node<K,V> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }

    boolean isMarker() {
        return value == this;
    }

    boolean isBaseHeader() {
        return value == BASE_HEADER;
    }

    boolean appendMarker(Node<K,V> f) {
        return casNext(f, new Node<K,V>(f));
    }

    void helpDelete(Node<K,V> b, Node<K,V> f) {
        if (f == next && this == b.next) {
            if (f == null || f.value != f)
                appendMarker(f);
            else
                b.casNext(this, f.next);
        }
    }

    V getValidValue() {
        Object v = value;
        if (v == this || v == BASE_HEADER)
            return null;
        return (V)v;
    }

    AbstractMap.SimpleImmutableEntry<K,V> createSnapshot() {
        V v = getValidValue();
        if (v == null)
            return null;
        return new AbstractMap.SimpleImmutableEntry<K,V>(key, v);
    }


    private static final sun.misc.Unsafe UNSAFE;
    private static final long valueOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class k = Node.class;
            valueOffset = UNSAFE.objectFieldOffset
                    (k.getDeclaredField("value"));
            nextOffset = UNSAFE.objectFieldOffset
                    (k.getDeclaredField("next"));
            } catch (Exception e) {
                throw new Error(e);
            }
    }
}
```

简单看下来，最大的感触自然就是全程没有出现synchronized或Lock，看来又是一个只依赖CAS实现并发控制的线程安全的容器(之所以说又，是因为我想到了[Java 并发-ConcurrentLinkedQueue](/2018/02/05/Java 并发-ConcurrentLinkedQueue/))

除了Node外，ConcurrentSkipListMap中另一个重要的数据结构名为Index。它也是ConcurrentSkipListMap的静态成员内部类，类定义如下：

```
static class Index<K,V>
```

Index负责将各层链表拼接起来，它有如下关键的实例成员变量：

```
// Index内部封装的节点
final Node<K,V> node;

// 向下的引用
final Index<K,V> down;

// 向右的引用
volatile Index<K,V> right;
```

也就是说，从外部看，ConcurrentSkipListMap中存储的基本元素是Index，它负责进行网络的构建，其内部封装着存储实际业务逻辑的Node。

此外，每一层链表的表头Index中还要存储本行链表是第几层。ConcurrentSkipListMap将这种表头Index定义为它的静态成员内部类HeadIndex。既然是特殊的Index，那么HeadIndex自然是Index的子类。该类的代码很短，全部代码如下：

```
static final class HeadIndex<K,V> extends Index<K,V> {
    final int level;
    HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
        super(node, down, right);
        this.level = level;
    }
}
```