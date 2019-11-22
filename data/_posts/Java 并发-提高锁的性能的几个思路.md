---
title: Java 并发-提高锁的性能的几个思路
date: 2018-04-27 14:59:36
tags: [Java,并发,锁]
categories: Java 并发
---

"锁"是最常用的同步策略之一。在高并发的环境下，激烈的锁竞争会导致程序的性能下降。所以锁的性能的优化是一个很值得探讨的话题，例如：避免死锁，减小锁粒度，锁分离等。

<!-- more -->

在多核机器中，较之单线程串行执行，使用多线程可以显著提高系统的性能，却也会增加额外的系统开销。对于单任务或者单线程的应用而言，其资源消耗基本都花费在任务本身上：它既不需要维护并行数据结构间的一致性状态，也不需要为线程的切换及调度花费时间。但对多线程的应用而言，除了需要满足任务本身的需求，还需要额外维护多线程环境的特有信息，例如：线程本身的元数据，线程的调度，线程上下文的切换等。正因为如此，在单核CPU上，采用并行算法的效率通常要低于对应的串行算法。

而对同步策略(也就是本文要探讨的锁)的优化，最重要的目标也就是优化这一部分因引入多线程而增加的"额外操作"，例如进行更合理的任务调度，更充分的压榨每个CPU的性能等等。本文将从比较高的维度介绍几个常用的优化思路。

**注意：**本文提到的"锁"是广义上的，即同时包含了Lock及synchronized。

# 减少锁的持有时间

无论如何优化加锁操作本身，只要进行加锁就会产生"额外"的系统消耗，那么尽可能的缩小需要加锁的区域，减少每个线程的持锁时间，就是最自然，也是最容易想到的优化思路。

具体来说，我们可以看如下代码：

```
public synchronized void syncMethod() {
    m1();    // 无需同步控制
    syncM();    // 需同步控制
    m2();    // 无需同步控制
}
```

很显然，线程在执行m1()及m2()时是无需加锁的。很自然的，我们就可以想到如下优化策略：

```
public void syncMethod() {
    m1();    // 无需同步控制
    synchronized(this) {
        syncM();    // 需同步控制   
    }
    m2();    // 无需同步控制
}
```

JDK的源码中有很多地方都应用了这种优化策略，例如处理正则表达式的java.util.regex.Pattern中就有如下方法：

```
public Matcher matcher(CharSequence input) {
    if (!compiled) {    // 只有在表达式未编译时，才会进行局部的加锁
        synchronized(this) {
            if (!compiled)
                compile();
        }
    }
    Matcher m = new Matcher(this, input);
    return m;
}
```

关于这个方法，有一个很有趣的点:if (!compiled)被连续判断了两次。做第一次判断时并未进行并发控制，无额外的开销。而只要进入了同步代码块，就是线程安全的了。但是我们无法保证在第一次判断至进入同步代码块期间(当然，仅从代码来看，二者是紧邻着的)不会发生同步问题：例如恰好有另一个线程利用这段时间完成了一次完成的编译操作。因此，我们有必要在进入同步代码块后再进行一次判断。

那么为什么不在第一次判断外面就包一层同步代码块呢？这样不就只需要一次判断了吗？这就是本小节优化思路的应用了：若if (!compiled) == false，也就是已经编译完了，无需再编译了，那么此时是不需要进行同步控制的，而很显然，在字符串匹配的整个周期中，编译只需要1次，绝大多数时候，当我们调用matcher(CharSequence input)方法时，字符串都是编译好的，因此我们需要缩小需要加锁的范围：只有当确实没编译时才执行加锁操作。这是本思路下比较高级的优化了：加锁范围缩窄的不是具体的代码，而是不同的条件分支。

# 减小锁粒度

在JDK中，本思路的典型应用是java.util.concurrent.ConcurrentHashMap，其类定义如下：

```
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V> implements ConcurrentMap<K, V>, Serializable
```

作为一个Map，最重要也是最常用的两个方法自然就是get()与put()了。而ConcurrentHashMap作为一个线程安全的容器，这种修改操作自然是要进行并发控制的。而最容易想到的并发策略自然就是对整个容器加锁。但这样做，我们就认为加锁的粒度太大了：因此ConcurrentHashMap在内部进一步的又被分为若干个小的Map，称之为段(Segment)，默认情况下，一个ConcurrentHashMap会被分为16段(额外多说一句，这种分段加锁的思想的应用还有[Java 并发-ConcurrentSkipListMap](/2018/02/07/Java 并发-ConcurrentSkipListMap/))。

如果要在ConcurrentHashMap中增加一个新的元素，并不需要对整个容器加锁，而是首先根据元素的hashcode值计算出该元素应该被存放在哪个段中，然后只对该段加锁，随后完成put()操作。这样，在并发环境中，如果有多个线程同时进行put()操作，只要被加入的元素不在同一个段中，就无需进行并发控制。由于默认有16个段，那么ConcurrentHashMap最多可供16个线程同时插入。

下面我们贴出put()方法的具体代码：

```
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject
         (segments, (j << SSHIFT) + SBASE)) == null)
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}
```

但是，减小锁粒度会引入一个新的问题：当系统需要取得容器的全局锁时，反而会变得麻烦。具体来说，如果要将整个容器都锁住，那么就需要将所有段的锁都拿在手里才行。比如说size()方法：

```
public int size() {
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow;
    long sum;
    long last = 0L;
    int retries = -1;
    try {
        for (;;) {
            if (retries++ == RETRIES_BEFORE_LOCK) {    // 为所有段加锁
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock();
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {    // 为所有段解锁
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size;
}
```

通常我们看到的JDK源码中的容器的size()方法，往往只有几句代码，逻辑也很清晰。然而ConcurrentHashMap的size()显然就很复杂了。要获得所有段的锁后才能实际开始求值。单就size()方法而言，ConcurrentHashMap在并发环境下的性能甚至是还不如由java.util.Collections.synchronizedMap()方法得到的线程安全的Map的。

这给了我们一个启示：对于ConcurrentHashMap这种分段加锁的容器而言，其调用需要全局锁的方法时的性能往往不如全局加锁控制线程安全的容器。因此选择并发容器时还应该看场景才行。

# 使用读写分离锁

在JDK中，本思路的典型应用是[Java 并发-读写锁ReadWriteLock](/2018/01/08/Java 并发-读写锁ReadWriteLock/)

事实上，读写分离锁这种优化思路可以看作是减小锁粒度这一优化思路的特例：上文中提到的ConcurrentHashMap是从数据的角度减小锁粒度，而使用读写分离锁则是从功能的角度上减小锁粒度。

# 锁分离

锁分离是读写分离锁的扩展，因此也可看作是减小锁粒度的特例。读写分离锁划分功能的依据是"读与写"，那么我们同样可以依据其他功能来划分。比较典型的例子就是java.util.concurrent.LinkedBlockingQueue，其类定义如下：

```
public class LinkedBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable
```

它是[Java 并发-BlockingQueue](/2018/02/07/Java 并发-BlockingQueue/)的具体实现，在这篇文章中，我们讨论了LinkedBlockingQueue的近亲ArrayBlockingQueue，现在我们再来看看LinkedBlockingQueue。

作为BlockingQueue，最重要也是最常用的两个方法自然是take()及put()，它们分别代表阻塞式的读及写。因为LinkedBlockingQueue底层是以链表实现的，那么take()为弹出队首，put()则为向队尾追加新值。若使用独占锁，那么这两个操作是无法同时进行的。但事实上，这两个操作可以并行进行，彼此间并不会产生冲突。

具体来说，LinkedBlockingQueue将独占锁一分为二，分别用于take()及put()。LinkedBlockingQueue中有如下实例成员：

```
private final ReentrantLock takeLock = new ReentrantLock();

private final Condition notEmpty = takeLock.newCondition();

private final ReentrantLock putLock = new ReentrantLock();

private final Condition notFull = putLock.newCondition();
```

自然，相应的Condition会绑在对应的锁上：take()时需等待notEmpty，put()时则需等待notFull。

然后我们具体来看下take()的代码：

```
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}

private void signalNotFull() {
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        notFull.signal();
    } finally {
        putLock.unlock();
    }
}
```

再来看下put()：

```
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    int c = -1;
    Node<E> node = new Node(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        while (count.get() == capacity) {
            notFull.await();
        }
        enqueue(node);
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}

private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}
```

基本思路与ArrayBlockingQueue一致。不过由于LinkedBlockingQueue有两把不同的锁，两个Condition条件则是分别绑在对应的锁上，导致要发另一个锁的Condition条件时不大方便：需要再加锁解锁一次(也就是上例中的signalNotFull()及signalNotEmpty())。因此，在take()方法中，若在本次拿取后依然还有元素，则会发送一个notEmpty，供其他take()使用。同理，在put()方法中，若在本次添加后仍未到达最大容量，则会发送一个notFull，供其他put()使用。

# 锁粗化

我们提出的第一个锁优化策略为"减少锁的持有时间"，其目的旨在尽可能的压缩每个线程占有锁的时间。然而有趣的是，我们本小节意欲介绍的优化策略正好站在它的反面：即适当延长线程对锁的占有时间。其缘由就在于加锁解锁是需要消耗系统资源的，如果加锁解锁的操作较为频繁，而实际的加锁后的业务时间又不是很长，这就导致系统用于加锁解锁调度切换的时间比重较之实际运行业务代码的时间增大，若该值大到一定程度，还不如不要释放锁，一直持有到阶段性任务完结得好。

其实，JVM会依据一定的算法，隐式的帮助我们完成部分的锁粗化操作：例如，在JVM遇到一连串的对同一个锁的加锁解锁操作时，便可能会将所有的锁操作整合为对锁的一次请求，例如如下代码：

```
public void m() {
    synchronized (this) {
        // do sth1
    }
    // 不需要并发控制的代码，很快能完成
    synchronized (this) {
        // do sth2
    }
}
```

则其可能会被整合为：

```
public void m() {
    synchronized (this) {
        // do sth1
        // 不需要并发控制的代码，很快能完成
        // do sth2
    }
}
```

写代码时，我们也应有意识的判断是否应进行锁的粗化。例如如下代码：

```
for (int i = 0; i < 1000; i++) {
    synchronized (this) {
        i++;
    }
}
```

这段代码会在每次for循环内部进行一次加锁解锁，而每次循环实际的业务仅仅是简单的i++，显然这是很不合理的，因此改为：

```
synchronized (this) {
    for (int i = 0; i < 1000; i++) i++;
}
```

这样就会合理得多。