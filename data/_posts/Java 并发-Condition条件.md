---
title: Java 并发-Condition条件
date: 2017-12-29 15:11:36
tags: [Java,并发,锁,Lock,ReentrantLock,重入锁,Condition]
categories: Java 并发
---

synchronized通过wait()及notify()实现线程间的通信。java.util.concurrent.locks.Lock接口号称synchronized的功能升级版，自然也应实现相应的通信机制。该机制被封装在java.util.concurrent.locks.Condition接口的实现类中。

Lock接口中有如下方法：

```
Condition newCondition();
```

该方法会返回一个Condition接口某实现类的实例，该实例将与调用它的锁实例绑定，用于该锁实例的通信。

<!-- more -->

Condition接口的全部方法如下：

```
/**
 * 使当前线程等待，并放弃已获得的锁
 * 会因signal()或signalAll()停止等待并重新尝试获取锁
 * 会因线程中断而结束等待
 */
void await() throws InterruptedException;

/**
 * 不响应中断，其余与await()相同
 */
void awaitUninterruptibly();

long awaitNanos(long nanosTimeout) throws InterruptedException;

boolean await(long time, TimeUnit unit) throws InterruptedException;

boolean awaitUntil(Date deadline) throws InterruptedException;

/**
 * 随机唤醒一个处于await状态的线程
 */
void signal();

/**
 * 唤醒所有处于await状态的线程
 */
void signalAll();
```

较之synchronized机制，很显然，await对应于wait，signal对应于notify。

下面来看一个小例子：

```
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Test {
    private static ReentrantLock LOCK = new ReentrantLock();

    private static Condition CONDITION = Test.LOCK.newCondition();

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                Test.LOCK.lock();
                try {
                    Test.CONDITION.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("after wait");
                Test.LOCK.unlock();
            }
        };
        new Thread(r).start();
        Thread.sleep(100);
        Test.LOCK.lock();
        Test.CONDITION.signal();
        Test.LOCK.unlock();
    }
}
```

输出：

```
after wait
```

显然，套路与synchronized极其相似，只不过synchronized用作同步标志的是任意对象，通讯机制是任意对象都有的wait和notify方法，而锁机制则做了特化：同步对象是Lock，通讯机制被封装在与该Lock对象绑定的Condition中。

在synchronized中，若要使用通讯机制，则必须先获得监视器对象的认可，在synchronized代码块的范围内也只能调用监视器对象的wait和notify方法。同理，若要使用锁的通讯机制，必须先获得锁的钥匙(lock)，而后在持有钥匙期间(unlock之前)调用与锁绑定的Condition对象的await及signal方法。

不过，需要注意的是，synchronized是用一对{}区分同步区域的，离开该区域自动退出同步。而锁机制则需要显式加锁及解锁，在提高灵活性的同时也增大了程序员犯错的可能：很有可能只顾着加锁而忘记了解锁。例如我们按如下方式修改上例：

```
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Test {
    private static ReentrantLock LOCK = new ReentrantLock();

    private static Condition CONDITION = Test.LOCK.newCondition();

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                Test.LOCK.lock();
                try {
                    Test.CONDITION.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("after wait");
                Test.LOCK.unlock();
            }
        };
        new Thread(r).start();
        Thread.sleep(100);
        Test.LOCK.lock();
        Test.CONDITION.signal();
	// 没有释放锁
        // Test.LOCK.unlock();
    }
}
```

此时程序无输出且无法结束。

对于synchronized而言，每个监视器对象只能对应一个通讯队列。而在锁机制中，每个Lock对象可绑定多个Condition，功能自然更为强大。

Java API中大量使用锁及其通讯机制，以java.util.concurrent.ArrayBlockingQueue为例。其首先按如下方式声明了锁：

```
final ReentrantLock lock;

private final Condition notEmpty;    // 信号量：非空

private final Condition notFull;    // 信号量：非满
```

构造函数中完成了锁及信号量的初始化：

```
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
```

关于锁的运用我们来看两个典型的方法，向队尾追加put()及从队头取出take()。

首先来看put()：

```
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            // 当前队列已满，等待收到非满的信号时才能继续
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
    notEmpty.signal();    // 因为放入了一个元素，发出非空的信号
                          // 如果当前有线程正在等待该信号则可继续执行下去
}
```

相应的take()为：

```
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            // 当前队列为空，等待收到非空的信号时才能继续
            notEmpty.await();
        return extract();
    } finally {
        lock.unlock();
    }
}

private E extract() {
    final Object[] items = this.items;
    E x = this.<E>cast(items[takeIndex]);
    items[takeIndex] = null;
    takeIndex = inc(takeIndex);
    --count;
    notFull.signal();    // 因为取出了一个元素，发出非满的信号
                         // 如果当前有线程正在等待该信号则可继续执行下去
    return x;
}
```

很显然，这就是操作系统中典型的多生产者(通过put放入缓冲区)-单缓冲区(队列对象)-多消费者(通过take从缓冲区中取出)模型。