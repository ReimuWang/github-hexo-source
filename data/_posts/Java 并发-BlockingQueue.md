---
title: Java 并发-BlockingQueue
date: 2018-02-07 14:43:36
tags: [Java,并发]
categories: Java 并发
---

在工作中，我们经常会遇到生产者-消费者模型的应用，例如：如果两个系统需要进行消息的沟通，我们往往会使用某个基于AMQP(Advanced Message Queuing Protocol)协议的消息队列，例如RabbitMQ,Qpid等。

而如果是在一个程序的内部，两个模块间需要线程安全的通信又该怎么做呢？

Java API为我们提供了java.util.concurrent.BlockingQueue接口，它的接口定义如下：

```
public interface BlockingQueue<E> extends Queue<E>
```

<!-- more -->

它是一个线程安全的，高性能的阻塞队列。通常被用于模块间消息通讯的缓冲区。Java API为我们提供了很多实现，最常用的是如下两个：

java.util.concurrent.ArrayBlockingQueue，类定义如下：

```
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable
```

java.util.concurrent.LinkedBlockingQueue，类定义如下：

```
public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable
```

顾名思义，ArrayBlockingQueue底层以数组实现，而LinkedBlockingQueue底层则以链表实现。二者的优缺点比较基本也就是数组与链表的优缺点比较。因此，ArrayBlockingQueue常用做有界队列，这样就避免了因扩展数组而造成的性能损失。而LinkedBlockingQueue常做无界队列，不会因初始值过大而在一开始便吃掉系统大量的内存。

而BlockingQueue之所以能作为阻塞队列，其关键还在于Blocking。而所有的阻塞队列都需要考虑一个基本的问题：当读线程消费完当前队列中所有的消息后，它如何得知下一条消息何时到来呢？当写线程因阻塞队列已满而无法写入新消息时，它如何知道何时才会有空间写入呢？

最为简单粗暴的方式就是让读写线程在空闲时不断的轮询阻塞队列，这样做虽然理论上可行，但显然造成了很多不必要的性能损失，而且轮训的间隔周期也不好确定。

BlockingQueue则是在队列为空时让读线程等待，待有消息进入阻塞队列后再将其唤醒。反之写线程也是同理。那么它是如何实现的呢？

我们不妨以ArrayBlockingQueue为例来说明这个问题。

顾名思义，ArrayBlockingQueue的底层数据结构自然是一个数组，它是ArrayBlockingQueue的一个实例成员变量：

```
final Object[] items;
```

向队列插入数据常用offer()方法或put()方法。我们先来说较为简单的offer()方法：

```
public boolean offer(E e) {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count == items.length)
            return false;
        else {
            insert(e);
            return true;
        }
    } finally {
        lock.unlock();
    }
}
```

该方法定义在Queue接口中，而ArrayBlockingQueue的实现也是在保证线程安全的基础上完全遵循了Queue接口的规范：若插入成功返回true，若因容量已满插入失败则立刻返回false。并未起到阻塞的作用。它的代码逻辑也很简单清晰，就不赘述了。

那么，实现阻塞的大任自然是着落在另一个插入方法put()上了。该方法插入成功当然也是立刻返回true，因容量已满而导致插入失败时则会让写线程一直等待，直到阻塞队列倒出空间。

同理，弹出队首元素常用poll()方法或take()方法。poll()方法的代码为：

```
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return (count == 0) ? null : extract();
    } finally {
        lock.unlock();
    }
}
```

同offer()方法，poll()方法依然是声明于Queue接口中，而ArrayBlockingQueue除了将其线程安全化之外依然没做额外的操作：若队列不为空则返回队首元素，反之则立刻返回null。

而take()方法就是那个阻塞的弹出方法了：若队列不为空则返回队首元素，反之则让读线程一直等待，直到队列非空。

为了实现put()及take()方法的功能，ArrayBlockingQueue在它的锁对象上绑定了两个Condition：

```
final ReentrantLock lock;

private final Condition notEmpty;

private final Condition notFull;
```

看到这里我想大家已经能隐隐猜到Doug Lea的套路了：当读线程调用take()方法时，如果队列为空，则让读线程等待notEmpty。同理，当写线程调用put()方法时，若队列已满，则让写线程等待notFull。

那么到底是不是这样呢？我们赶紧来看看take()方法的源码吧：

```
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return extract();
    } finally {
        lock.unlock();
    }
}
```

其中extract()方法是在确保了非空后的取消息操作：

```
private E extract() {
    final Object[] items = this.items;
    E x = this.<E>cast(items[takeIndex]);
    items[takeIndex] = null;
    takeIndex = inc(takeIndex);
    --count;
    notFull.signal();
    return x;
}
```

除了正常的取值操作外，最只得我们关注的就是notFull.signal();这一句了：因为取了一个元素，便空出了一个位子，自然可以发一个非空的信号了。看了一个信号对应一个元素。

进一步的，我们也不难推测出，put()方法因为放入了一个元素，队列自然就非空了，那么应该会释放一个notEmpty.signal();才对。那么是不是这样呢，我们再来看put()方法的源码：

```
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        insert(e);
    } finally {
        lock.unlock();
    }
}

private void insert(E x) {
    items[putIndex] = x;
    putIndex = inc(putIndex);
    ++count;
    notEmpty.signal();
}
```

哦耶！推测完全正确。