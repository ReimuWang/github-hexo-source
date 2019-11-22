---
title: Java 并发-CopyOnWriteArrayList
date: 2018-02-07 11:35:36
tags: [Java,并发]
categories: Java 并发
---

很多时候，我们对某些容器的操作确实是并发的，然而读却远多于写。这其实是一个比较尴尬的局面：因为确实存在并发的写操作，因此必须设置线程安全策略，然后由于读远多于写，该策略登场的机会不高，毕竟纯粹是读的话即便是并发环境也是不需要容器是线程安全的。

<!-- more -->

因此，操作系统中提出了读写锁的概念，算是针对这种情况对普通的锁进行了特化。简单来说：

- 情况1：若已有读者，则新的读者不会被阻塞。

- 情况2：若已有读者，则新的写者会被阻塞。

- 情况3：若已有写者，则新的读者会被阻塞。

- 情况4：若已有写者，则新的写者会被阻塞。

实际上，只有情况1得到了优化。不过由于应用场景本就是读远多于写，因此对性能的提升还是巨大的。

Java API中的java.util.concurrent.CopyOnWriteArrayList也提供了类似于读写锁的功能。不过它更进一步，不仅是情况1，情况2,3也不会被阻塞了。也就是阻塞范围被缩小为只有写者-写者。

那么它是如何做到的呢？玄机其实就藏在它的类名中：CopyOnWrite。也就是在写操作前，会进行一次自我复制，这是典型的用空间换时间的做法。

具体来说，当CopyOnWriteArrayList需要被修改时，我们并不会修改原内容(这就保证了读线程的数据一致性)，而是生成一份源数据的副本，将修改都作用到这份副本上，而后再在合适的时机用修改后的副本替换源数据。

CopyOnWriteArrayList的类定义如下：

```
public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

和预想的稍有不同，看名字还以为它是ArrayList的子类，结果二者并没有直接的联系。不过既然CopyOnWriteArrayList的名称中包含ArrayList，那么它底层用于存储数据的结构就应该是一个数组。事实上也确然如此。CopyOnWriteArrayList中有如下成员变量：

```
private volatile transient Object[] array;
```

所谓的读写，实际上就是在折腾这个数组。读操作举例如下：

```
public E get(int index) {
    return get(getArray(), index);
}
```

这是最常见的以索引index取元素的方法，它调用了两个方法：

```
final Object[] getArray() {
    return array;
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
```

这些方法的逻辑都非常简单，也没有任何的线程安全策略。

相对来说，写操作就要复杂一些了：

```
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        // 生成一份副本，为容纳新元素，副本容积+1
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        // 用副本替换源数据
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

使用了重入锁ReentrantLock做并发控制，总体来说，逻辑还是很清晰的(相对于[Java 并发-ConcurrentLinkedQueue](/2018/02/05/Java 并发-ConcurrentLinkedQueue/)这种使用无锁CAS的容器而言)。

本以为其中的setArray()方法里面会有什么牛逼的套路，点开之后却发现非常简单：

```
final void setArray(Object[] a) {
    array = a;
}
```

稍微有点不理解大神是怎么想的，这么简单的代码为啥不直接写，还要在外面包一个方法？

最后还需要特别声明的一点就是array是被volatile修饰的，这样才能保证写操作对a的替换修改能立刻为读操作所见。