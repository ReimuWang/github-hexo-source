---
title: Java 并发-无锁
date: 2018-05-11 19:16:36
tags: [Java,并发,无锁]
categories: Java 并发
---

在[Java 并发-并发级别](/2017/10/04/Java 并发-并发级别/)中，我们曾对无锁有过简要的介绍。本文将对其做更为具体的讲解。

<!-- more -->

# CAS

无锁是一种非阻塞的并发级别。它所采用的策略为比较交换(Compare And Swap, CAS)。

简单来说，CAS的模型是这样的：

```
CAS(V, E, N)
```

其中：

- V: 欲更新的变量
- E: 对欲更新变量的预期值
- N: 对欲更新变量要赋的新值

换句话说，V是变量，而E,N是V可能为的值。若欲将V赋值为N，会在赋值之前检查V当前的值，当且仅当当前值仍为E时，即说明并未有其他线程修改V时，才会将V赋值为N。否则将什么都不做。然后无论此前做了什么，CAS都会返回V的当前值。

其实更较真的来说，当前值仍为E其实并不能说明在此期间就没有其他线程修改V。很有可能的是，V的值经历了很多次的修改，恰好在该线程检查时值又被赋回了E。不过这其实并不重要：因为我们只关注结果，只要在线程需要修改V时，它的值仍是期望值即可。

至于如果修改失败，线程打算做什么，那就不是CAS关注的事情了，会完全交给线程自身去判断：如果线程接受失败，那么就放弃这次并发操作。反之，它就再次尝试，直到成功为止。

在硬件层面，大部分的现代处理器都会支持原子化的CAS指令。自JDK1.5起，JVM会直接使用这些指令来实现部分的并发操作和并发数据结构。很显然，这样做的性能是远高于使用锁及同步的。举个小例子，我们在[Java 并发-ConcurrentLinkedQueue](/2018/02/05/Java 并发-ConcurrentLinkedQueue/)中介绍过的ConcurrentLinkedQueue就是一个基于无锁实现的线程安全的容器。在此我们不妨再次贴出其中核心类Node的代码：

```
private static class Node<E> {
    // item,next支撑起了Node作为链表的节点的基础
    volatile E item;    // 节点中存储的数据值
    volatile Node<E> next;    // 下一个节点

    Node(E item) {
        NSAFE.putObject(this, itemOffset, item);
    }

    /**
     * 设置节点中存储的数据值
     * 利用CAS保证线程安全性：
     * 设置时当前值与期望值相同则设置为目标值
     * @param cmp E, 期望值
     * @param val E, 目标值
     * @return boolean, true--设置成功，false--设置失败
     */
    boolean casItem(E cmp, E val) {
        return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
    }

    void lazySetNext(Node<E> val) {
        UNSAFE.putOrderedObject(this, nextOffset, val);
    }

    /**
     * 设置本节点的下一个节点
     * 利用CAS保证线程安全性：
     * 设置时当前值与期望值相同则设置为目标值
     * @param cmp Node<E>, 期望值
     * @param val Node<E>, 目标值
     * @return boolean, true--设置成功,false--设置失败
     */
    boolean casNext(Node<E> cmp, Node<E> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }


    private static final sun.misc.Unsafe UNSAFE;
    private static final long itemOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class k = Node.class;
            itemOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("item"));
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

CAS操作在Java API中很常见，不过调用方式都是大同小异，核心围绕的都是UNSAFE.compareAndSwapObject()。这个方法是属于UNSAFE类的，也就是说它是不对外开放的，只供Java API内部使用。官方下载的JDK中也没有附带它的源码，不过我们可以通过它的接收参数以及功能很明显的看出这就是一个CAS。只不过它的返回值是boolean，看来方法内部对原生的CAS做了包装。其实这是更符合我们的调用思维的：调用者其实并不太关心当前V的值，而是更关心此次修改是否成功。

# AtomicInteger

为了让Java程序员能直接受益于CAS等CPU指令，JDK提供了如下包：

```
java.util.concurrent.atomic
```

该包中实现了一些直接使用CAS操作的线程安全的类型。下面我们介绍一下其中最常用的类：AtomicInteger。它的类定义如下：

```
public class AtomicInteger extends Number implements java.io.Serializable
```

简单来说，AtomicInteger可以被视为一个32位的整数，但与Integer不同的是，AtomicInteger是可变的，并且是线程安全的。它的任何并发操作，都是基于CAS指令进行的(也就是说AtomicInteger使用无锁进行并发控制)。我们先来简单概述一下AtomicInteger常用的方法，对于其他的原子类，操作也是非常类似的：

```
public final int get()    // 取得当前值
public final void set(int newValue)    // 设置当前值
public final int getAndSet(int newValue)    // 设置新值，并返回旧值
public final boolean compareAndSet(int expect, int update)    // 如果当前值为expect，则设置为update
public final int getAndIncrement()    // 当前值加1，返回旧值
public final int getAndDecrement()    // 当前值减1，返回旧值
public final int getAndAdd(int delta)    // 当前值增加delta，返回旧值
public final int incrementAndGet()    // 当前值加1，返回新值
public final int decrementAndGet()    // 当前值减1，返回新值
public final int addAndGet(int delta)    // 当前值增加delta，返回新值
```

AtomicInteger类中最核心的成员变量为：

```
private volatile int value;
```

value中存储的值就是AtomicInteger的当前实际取值。此外还有一个字段：

```
private static final long valueOffset;
```

该字段为value的偏移量，它是AtomicInteger进行并发控制的关键。

我们先来看一个小例子：

```
import java.util.concurrent.atomic.AtomicInteger;

public class Test {

    private static AtomicInteger AI = new AtomicInteger();

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10000; i++) Test.AI.incrementAndGet();
            }
        };
        int threadCount = 10;
        Thread[] threadArray = new Thread[threadCount];
        for (int i = 0; i < threadCount; i++) threadArray[i] = new Thread(r);
        for (int i = 0; i < threadCount; i++) threadArray[i].start();
        for (int i = 0; i < threadCount; i++) threadArray[i].join();
        System.out.println(Test.AI);
    }
}
```

输出：

```
100000
```

显然，AtomicInteger很好的进行了并发控制。接下来让我们来看一下incrementAndGet()的源码：

```
public final int incrementAndGet() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}
```

它内部调用的get()方法很简单：

```
public final int get() {
    return value;
}
```

类似于AtomicInteger，AtomicLong对应于long型，AtomicBoolean对应于boolean，AtomicReference对应于引用。

# Unsafe

接上一小节AtomicInteger，我们在分析它的incrementAndGet()方法的源码时并未看其中的核心方法compareAndSet()，现在我们具体的来看一下：

```
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

而unsafe是AtomicInteger的类变量：

```
private static final Unsafe unsafe = Unsafe.getUnsafe();
```

果然，它仅仅做了一个封装，其内部调用的依然是我们在介绍CAS时提到的UNSAFE类。

现在就让我们来具体介绍一下这个类吧。

如前文所述，JDK并不希望程序员直接调用Unsafe类，因此也没有提供它的源码。不过我们依然可以查到它所属的包为：

```
sun.misc
```

顾名思义，Unsafe类中应该是封装了一些不安全的操作(这样的话，不希望程序员们调用也就可以理解了)，那么什么是不安全的操作呢？

在 C/C++ 中，有一个很重要的数据结构，名为指针。众所周知，指针是不安全的，这也是Java不准程序员显式定义和使用指针的原因。

请注意上文中的措辞，Java只是不准程序员显式的定义及使用指针，并不意味着Java移除了指针。事实上，指针所实现的功能是无可替代的：我们总要用一个数据结构去指向变量。如果不是特别较真的话，我们可以把引用看作Java中的指针，然而引用是非常上层的东西了，它并不会真正导致指针会造成的那些错误：例如，数组越界。如果是指针的话，那么当真就是在系统底层指到了数组外面。而如果是引用，则是JVM内部的行为，它会在内部"计算"是否越界，如果越界了那么返回异常，并没有实际在底层进行操作，相当于只进行了一次预先演算。这样便规避了实际运行时可能会产生的风险。

然而，终归是要实际运行的。我们此前也提到，指针的功能是无可替代的。事实上，Java将指针大部分的功能封装在了Unsafe类中。换句话说，与引用相比，说Unsafe是Java的指针要更准确一些(仅仅只是更准确一些，Unsafe也无法完全代替指针)。

我们继续来分析compareAndSwapInt()。虽然官网上下载不到源码，但我们总是有搞到源码的渠道的：

```
/**
 * o 待操作对象
 * offset 对象内的偏移量
 *        是一个字段到对象头部的偏移量
 *        通过这个偏移量可以快速定位字段
 * expected 期望值
 * x 要设置的值
 *   若有当前值等于expected，则设置为x
 */
public final native boolean compareAndSwapInt(Object o, long offset, int expected, int x);
```

原来是native方法，不过这也与我们在前文中的推测相符：直接调用底层的CAS指令，无需再用Java代码做什么了。

此外，从名字上我们就不难推测出，compareAndSwapInt()方法是处理int型数据的，而我们在前文(开篇CAS那一小节)中提到的compareAndSwapObject()方法自然就是服务于Object对象的了。类比下去，常用的数据类型，long，double等应该也都会有特定的一族方法。

在此我们以int型为例，看看Unsafe还提供了哪些其他功能的方法：

```
// 获得o在偏移量offset上的字段的int值
public native int getInt(Object o, long offset);
// 设定o在偏移量offset上的字段的int值
public native void putInt(Object o, long offset, int x);
// 获得字段在对象中的偏移量
public native long objectFieldOffset(Field f);
// 使用volatile语义，设定o在偏移量offset上的字段的int值
public native void putIntVolatile(Object o, long offset, int x);
// 使用volatile语义，获得o在偏移量offset上的字段的int值
public native int getIntVolatile(Object o, long offset);
// 在putIntVolatile()的基础上，还要求被操作字段就是volatile的
public native void putOrderedInt(Object o, long offset, int x);
```

为了继续分析Unsafe类，我们不妨将上文中Node的代码再贴一遍：

```
private static class Node<E> {
    // item,next支撑起了Node作为链表的节点的基础
    volatile E item;    // 节点中存储的数据值
    volatile Node<E> next;    // 下一个节点

    Node(E item) {
        NSAFE.putObject(this, itemOffset, item);
    }

    /**
     * 设置节点中存储的数据值
     * 利用CAS保证线程安全性：
     * 设置时当前值与期望值相同则设置为目标值
     * @param cmp E, 期望值
     * @param val E, 目标值
     * @return boolean, true--设置成功，false--设置失败
     */
    boolean casItem(E cmp, E val) {
        return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
    }

    void lazySetNext(Node<E> val) {
        UNSAFE.putOrderedObject(this, nextOffset, val);
    }

    /**
     * 设置本节点的下一个节点
     * 利用CAS保证线程安全性：
     * 设置时当前值与期望值相同则设置为目标值
     * @param cmp Node<E>, 期望值
     * @param val Node<E>, 目标值
     * @return boolean, true--设置成功,false--设置失败
     */
    boolean casNext(Node<E> cmp, Node<E> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }


    private static final sun.misc.Unsafe UNSAFE;
    private static final long itemOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class k = Node.class;
            itemOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("item"));
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

其中，得到Unsafe类实例的代码在static代码块中：

```
UNSAFE = sun.misc.Unsafe.getUnsafe();
```

虽然官方不提供，但我们依然可以从其他渠道拿到它的源码：

```
public static Unsafe getUnsafe() {
    Class cc = Reflection.getCallerClass();
    if (cc.getClassLoader() != null)
        throw new SecurityException("Unsafe");
    return theUnsafe;
}
```

显然，该方法会检查调用者所属的类，如果该类的ClassLoader不为null，则说明该请求并非来自JDK内部，而正如前文中提到的，Unsafe类被设计成只供JDK内部调用，因此会抛出异常，拒绝响应。

关于能进行这样判断的依据，可详见[JVM-类加载器](/2017/12/04/JVM-类加载器/)。

# AtomicReference

从名字上我们就不难推测，AtomicReference与前文介绍的AtomicInteger即为类似。只不过AtomicInteger保证的是整数的线程安全性，而AtomicReference保证得则是普通的对象引用。

在开篇介绍CAS时，我们曾有如下论述：

其实更较真的来说，当前值仍为E其实并不能说明在此期间就没有其他线程修改V。很有可能的是，V的值经历了很多次的修改，恰好在该线程检查时值又被赋回了E。不过这其实并不重要：因为我们只关注结果，只要在线程需要修改V时，它的值仍是期望值即可。

这里说的"其实并不重要"指得是通常情况下，某些时候，该机制是会产生问题的。我们先来看下面的小例子：

```
import java.util.concurrent.atomic.AtomicReference;

public class Test {

    private static final int STANDARD = 20;

    private static AtomicReference<Integer> AR = new AtomicReference<Integer>(Test.STANDARD - 1);

    public static void main(String[] args) {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                String threadId = "[" + Thread.currentThread().getId() + "]";
                Integer value = Test.AR.get();
                if (value < Test.STANDARD) {
                    if (Test.AR.compareAndSet(value, value + Test.STANDARD)) {
                        System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改成功，修改后值=" + Test.AR.get());
                    } else {
                        System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改失败，此时值=" + Test.AR.get());
                    }
                } else
                    System.out.println(threadId + "当前值=" + value + ",大于等于" + Test.STANDARD + "，无需修改");
            }
        };
        t1.start();
        Thread t2 = new Thread() {
            @Override
            public void run() {
                String threadId = "[" + Thread.currentThread().getId() + "]";
                Integer value = Test.AR.get();
                if (value < Test.STANDARD) {
                    if (Test.AR.compareAndSet(value, value + Test.STANDARD)) {
                        System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改成功，修改后值=" + Test.AR.get());
                    } else {
                        System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改失败，此时值=" + Test.AR.get());
                    }
                } else
                    System.out.println(threadId + "当前值=" + value + ",大于等于" + Test.STANDARD + "，无需修改");
            }
        };
        t2.start();
    }
}
```

上面这段代码使用两个线程并发修改AR的值。如果AR的值小于20则会加上20。只会修改1次。运行后输出为：

```
[9]当前值=19,小于20，进行修改并修改成功，修改后值=39
[10]当前值=39,大于等于20，无需修改
```

代码是没有问题的。

现在我们再添加线程3，它会减少AR中的值：

```
import java.util.concurrent.atomic.AtomicReference;

public class Test {

    private static final int STANDARD = 20;

    private static AtomicReference<Integer> AR = new AtomicReference<Integer>(Test.STANDARD - 1);

    public static void main(String[] args) {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                String threadId = "[" + Thread.currentThread().getId() + "]";
                Integer value = Test.AR.get();
                if (value < Test.STANDARD) {
                    if (Test.AR.compareAndSet(value, value + Test.STANDARD)) {
                        System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改成功，修改后值=" + Test.AR.get());
                    } else {
                        System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改失败，此时值=" + Test.AR.get());
                    }
                } else
                    System.out.println(threadId + "当前值=" + value + ",大于等于" + Test.STANDARD + "，无需修改");
            }
        };
        t1.start();
        Thread t2 = new Thread() {
            @Override
            public void run() {
                String threadId = "[" + Thread.currentThread().getId() + "]";
                Integer value = Test.AR.get();
                if (value < Test.STANDARD) {
                    if (Test.AR.compareAndSet(value, value + Test.STANDARD)) {
                        System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改成功，修改后值=" + Test.AR.get());
                    } else {
                        System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改失败，此时值=" + Test.AR.get());
                    }
                } else
                    System.out.println(threadId + "当前值=" + value + ",大于等于" + Test.STANDARD + "，无需修改");
            }
        };
        t2.start();
        Thread t3 = new Thread() {
            @Override
            public void run() {
                String threadId = "[" + Thread.currentThread().getId() + "]";
                Integer value = Test.AR.get();
                while (true) {
                    if (Test.AR.compareAndSet(value, value - Test.STANDARD)) {
                        System.out.println(threadId + "当前值=" + value + ",成功减少" + Test.STANDARD + "，减少后值=" + Test.AR.get());
                        break;
                    }
                }
            }
        };
        t3.start();
    }
}
```

此时输出：

```
[9]当前值=19,小于20，进行修改并修改成功，修改后值=39
[10]当前值=39,大于等于20，无需修改
[11]当前值=39,成功减少20，减少后值=19
```

依然是没有问题的。

然后我们有意的控制一下这3个线程执行过程中的耗时：

```
import java.util.concurrent.atomic.AtomicReference;

public class Test {

    private static final int STANDARD = 20;

    private static AtomicReference<Integer> AR = new AtomicReference<Integer>(Test.STANDARD - 1);

    public static void main(String[] args) {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                try {
                    String threadId = "[" + Thread.currentThread().getId() + "]";
                    Integer value = Test.AR.get();
                    Thread.sleep(10);
                    if (value < Test.STANDARD) {
                        if (Test.AR.compareAndSet(value, value + Test.STANDARD)) {
                            System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改成功，修改后值=" + Test.AR.get());
                        } else {
                            System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改失败，此时值=" + Test.AR.get());
                        }
                    } else
                        System.out.println(threadId + "当前值=" + value + ",大于等于" + Test.STANDARD + "，无需修改");
                } catch (Exception e) {}
            }
        };
        t1.start();
        Thread t2 = new Thread() {
            @Override
            public void run() {
                try {
                    String threadId = "[" + Thread.currentThread().getId() + "]";
                    Integer value = Test.AR.get();
                    Thread.sleep(10);
                    if (value < Test.STANDARD) {
                        Thread.sleep(100);
                        if (Test.AR.compareAndSet(value, value + Test.STANDARD)) {
                            System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改成功，修改后值=" + Test.AR.get());
                        } else {
                            System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改失败，此时值=" + Test.AR.get());
                        }
                    } else
                        System.out.println(threadId + "当前值=" + value + ",大于等于" + Test.STANDARD + "，无需修改");
                } catch (Exception e) {}

            }
        };
        t2.start();
        Thread t3 = new Thread() {
            @Override
            public void run() {
                try {
                    Thread.sleep(20);
                    String threadId = "[" + Thread.currentThread().getId() + "]";
                    Integer value = Test.AR.get();
                    while (true) {
                        if (Test.AR.compareAndSet(value, value - Test.STANDARD)) {
                            System.out.println(threadId + "当前值=" + value + ",成功减少" + Test.STANDARD + "，减少后值=" + Test.AR.get());
                            break;
                        }
                    }
                } catch (Exception e) {}
            }
        };
        t3.start();
    }
}
```

输出为：

```
[9]当前值=19,小于20，进行修改并修改成功，修改后值=39
[11]当前值=39,成功减少20，减少后值=19
[10]当前值=19,小于20，进行修改并修改成功，修改后值=39
```

现在便有问题了，AR的值被修改了两次。用语言描述的话，其执行顺序是下面这样的：

1. t1获得AR当前值=19，小于20，判断需要增加20。
2. t2获得AR当前值=19，小于20，判断需要增加20。
3. t1成功增加AR，增加后值为39。
4. t3将AR的值减少20，减少后值变回19。
5. t2开始进行CAS，因为当前值与期望值相同，故再次增加AR，增加后值为39。

如上所述，AR的值被增加了两次，变成了脏值。

上面的小例子是在控制了苛刻的执行顺序后产生的特例，一般不会发生。但这并不代表着它一定就不会发生。而且这种错误一旦发生，排查起来也是非常困难。因此依然有必要为此找到一个解决策略。而这个解决策略就是下面我们马上要介绍的AtomicStampedReference。

# AtomicStampedReference

继续上一节，其实并不仅仅是AtomicReference，绝大多数Atomic这一族的类都会有相同的并发问题。归根结底，还是因为这是CAS算法本身的问题。简单来说，这个问题就是：CAS只能判断进行比较时的那一个时间点上的当前值是否与期望值相同，而并非真正的知道在此期间值是否发生了变化。不过既然知道了症结所在，那么解决策略也就很显然了：用某种手段记录下值的变化情况即可。

不过，虽说是要记录值的变化情况，其实也并非就一定是做类似下面这样的详细记录：

- 线程1将V改为1
- 线程2将V改为2
- 线程3将V改为3
...

事实上，我们只需要知道在此期间是否有其他线程修改过变量即可。至于它们到底将它改成了什么样子，其实并不重要：因为只要知道已被修改过就足以判断这次修改失败了，不用再需要其他的信息了。

AtomicStampedReference就是Java API给出的解决策略，它的内部不仅维护了对象值，还维护了一个时间戳(并非严格意义上的时间戳，而是一个任意的可以记录修改状态的整数)。若修改成功，不仅仅会修改AtomicStampedReference的值，还需要修改它的时间戳。同理，在进行修改的CAS尝试时，不仅会判断对象值是否满足期望，还会判断时间戳是否满足期望。

较之AtomicReference，AtomicStampedReference增加了一些时间戳相关的方法：

```
/**
 * expectedReference, 期望值
 * newReference, 新值
 * expectedStamp, 期望时间戳
 * newStamp, 新时间戳
 */
public boolean compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp)

// 获得当前对象引用
public V getReference()

// 获得当前时间戳
public int getStamp()

// 设置当前对象引用和时间戳
public void set(V newReference, int newStamp)
```

现在，就让我们用AtomicStampedReference来重写上文中使用AtomicReference时出现错误的代码吧：

```
import java.util.concurrent.atomic.AtomicStampedReference;

public class Test {

    private static final int STANDARD = 20;

    // 构造函数传入的两个参数依次为：初始值，初始时间戳
    private static AtomicStampedReference<Integer> ASR = new AtomicStampedReference<Integer>(Test.STANDARD - 1, 0);

    public static void main(String[] args) {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                try {
                    String threadId = "[" + Thread.currentThread().getId() + "]";
                    Integer value = Test.ASR.getReference();
                    int timestamp = Test.ASR.getStamp();
                    Thread.sleep(10);
                    if (value < Test.STANDARD) {
                        if (Test.ASR.compareAndSet(value, value + Test.STANDARD, timestamp, timestamp + 1)) {
                            System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改成功，修改后值=" + Test.ASR.getReference());
                        } else {
                            System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改失败，此时值=" + Test.ASR.getReference());
                        }
                    } else
                        System.out.println(threadId + "当前值=" + value + ",大于等于" + Test.STANDARD + "，无需修改");
                } catch (Exception e) {}
            }
        };
        t1.start();
        Thread t2 = new Thread() {
            @Override
            public void run() {
                try {
                    String threadId = "[" + Thread.currentThread().getId() + "]";
                    Integer value = Test.ASR.getReference();
                    int timestamp = Test.ASR.getStamp();
                    Thread.sleep(10);
                    if (value < Test.STANDARD) {
                        Thread.sleep(100);
                        if (Test.ASR.compareAndSet(value, value + Test.STANDARD, timestamp, timestamp + 1)) {
                            System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改成功，修改后值=" + Test.ASR.getReference());
                        } else {
                            System.out.println(threadId + "当前值=" + value + ",小于" + Test.STANDARD + "，进行修改并修改失败，此时值=" + Test.ASR.getReference());
                        }
                    } else
                        System.out.println(threadId + "当前值=" + value + ",大于等于" + Test.STANDARD + "，无需修改");
                } catch (Exception e) {}

            }
        };
        t2.start();
        Thread t3 = new Thread() {
            @Override
            public void run() {
                try {
                    Thread.sleep(20);
                    String threadId = "[" + Thread.currentThread().getId() + "]";
                    Integer value = Test.ASR.getReference();
                    int timestamp = Test.ASR.getStamp();
                    while (true) {
                        if (Test.ASR.compareAndSet(value, value - Test.STANDARD, timestamp, timestamp + 1)) {
                            System.out.println(threadId + "当前值=" + value + ",成功减少" + Test.STANDARD + "，减少后值=" + Test.ASR.getReference());
                            break;
                        }
                    }
                } catch (Exception e) {}
            }
        };
        t3.start();
    }
}
```

输出：

```
[9]当前值=19,小于20，进行修改并修改成功，修改后值=39
[11]当前值=39,成功减少20，减少后值=19
[10]当前值=19,小于20，进行修改并修改失败，此时值=19
```

可以看到，值只被增加了1次，问题解决了。

# AtomicIntegerArray

Atomic一族中还有原子类性的容器，即原子数组，包括：AtomicIntegerArray,AtomicLongArray,AtomicReferenceArray。顾名思义，它们分别代表int型数组，long型数组及普通的对象数组。

这里我们以AtomicIntegerArray为例，介绍下原子数组的使用方式。

从本质上来讲，AtomicIntegerArray是对int[]的线程安全化封装。同其他Atomic一族的类一样，它的内部也是通过Unsafe类的CAS操作实现的。我们先简要介绍几个它的核心方法：

```
// 获得数组索引为i的元素
public final int get(int i)

// 获得数组的长度
public final int length()

// 将数组索引为i的元素设置为newValue，并返回旧值
public final int getAndSet(int i, int newValue)

// 进行CAS操作，若数组索引为i的元素等于期望值(expect)，则将之设置为新值(update)。设置成功后返回true，反之返回false
public final boolean compareAndSet(int i, int expect, int update)

// 将索引为i的元素加1
public final int getAndIncrement(int i)

// 将索引为i的元素减1
public final int getAndDecrement(int i)

// 将索引为i的元素加delta
public final int getAndAdd(int i, int delta)
```

下面我们来看一个小例子：

```
import java.util.concurrent.atomic.AtomicIntegerArray;

public class Test {

    public static void main(String[] args) throws InterruptedException {
        final int arrayLength = 10;
        final AtomicIntegerArray aia = new AtomicIntegerArray(arrayLength);
        Runnable r = new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 1000; i++) {
                    for (int j = 0; j < arrayLength; j++)
                        aia.getAndIncrement(j);
                }
            }
        };
        Thread[] threadArray = new Thread[arrayLength];
        for (int i = 0; i < arrayLength; i++) {
            threadArray[i] = new Thread(r);
            threadArray[i].start();
            threadArray[i].join();
        }
        System.out.println(aia);
    }
}
```

输出为：

```
[10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000]
```

符合预期。

最后我想说明的是。AtomicIntegerArray内实际用于存储数据的结构为：

```
private final int[] array;
```

这是一个被final修饰的变量。也就是说，从外观上看，AtomicIntegerArray除了具有线程安全性之外，与普通的数组基本是一致的：声明后自然也是不能动态扩容的。

# AtomicIntegerFieldUpdater

在[Java 并发-容器线程安全化方法](/2018/02/01/Java 并发-容器线程安全化方法/)中，我们介绍了将线程不安全的容器简单快速的改造为线程安全的容器的方法。同理，对于单独的变量，有时我们也需要能够将其简单的改造为线程安全的变量的方法。

Atomic一族中有一组以Updater结尾的类被用以解决这个需求，它们分别是AtomicIntegerFieldUpdater,AtomicLongFieldUpdater,AtomicReferenceFieldUpdater。顾名思义，它们会以CAS操作分别对int,long及普通对象进行线程安全化处理。

下面我们不妨以AtomicIntegerFieldUpdater为例，来看看具体的使用方法。

我们不妨模拟一个小场景：选举人参与竞选。只要投票人投了某选举人一票，该选举人即获得一分。投票结束后将统计各选举人获得的总分数。显然，这是一个身处并发环境下的问题，不过因为某些原因，得分字段并非是线程安全的，因此便需要使用AtomicIntegerFieldUpdater进行线程安全化处理。

代码如下：

```
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

public class Test {

    public static void main(String[] args) throws InterruptedException {
        final AtomicIntegerFieldUpdater<Candidate> aifu = AtomicIntegerFieldUpdater.newUpdater(Candidate.class, "score");
        // 检验AtomicIntegerFieldUpdater是否正确
        final AtomicInteger check = new AtomicInteger(0);
        final Candidate candidate = new Candidate();
        Thread[] threadArray = new Thread[10000];
        for (int i = 0; i < threadArray.length; i++) {
            threadArray[i] = new Thread() {
                @Override
                public void run() {
                    if (Math.random() > 0.4) {    // 约有60%的选民会投该候选人
                        aifu.incrementAndGet(candidate);
                        check.incrementAndGet();
                    }
                }
            };
            threadArray[i].start();
            threadArray[i].join();
        }
        System.out.println("AtomicIntegerFieldUpdater=" + candidate.score);
        System.out.println("AtomicInteger=" + check.get());
    }
}

class Candidate {
    // 得分并非是线程安全的
    volatile int score;
}
```

输出：

```
AtomicIntegerFieldUpdater=6104
AtomicInteger=6104
```

经过多次重复实验，结果均一致，符合预期。

虽然上文中已证明了AtomicIntegerFieldUpdater可以按照预期完成任务，不过它(其他的Updater也一样)依然存在着一些限制：

第一，本质上来说，Updater是使用反射来获得变量的，因此它只能修改它可见的变量。如果变量不可见，则会出错。例如，如果我们将上文代码中的score的访问权限设置为private，即：

```
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

public class Test {

    public static void main(String[] args) throws InterruptedException {
        final AtomicIntegerFieldUpdater<Candidate> aifu = AtomicIntegerFieldUpdater.newUpdater(Candidate.class, "score");
        // 检验AtomicIntegerFieldUpdater是否正确
        final AtomicInteger check = new AtomicInteger(0);
        final Candidate candidate = new Candidate();
        Thread[] threadArray = new Thread[10000];
        for (int i = 0; i < threadArray.length; i++) {
            threadArray[i] = new Thread() {
                @Override
                public void run() {
                    if (Math.random() > 0.4) {    // 约有60%的选民会投该候选人
                        aifu.incrementAndGet(candidate);
                        check.incrementAndGet();
                    }
                }
            };
            threadArray[i].start();
            threadArray[i].join();
        }
        System.out.println("AtomicIntegerFieldUpdater=" + candidate.getScore());
        System.out.println("AtomicInteger=" + check.get());
    }
}

class Candidate {
    // 得分并非是线程安全的
    private volatile int score;

    public int getScore() {
        return score;
    }
}
```

可以通过编译，不过输出时会报异常：

```
Exception in thread "main" java.lang.RuntimeException: java.lang.IllegalAccessException: Class com.test.Test can not access a member of class com.test.Candidate with modifiers "private volatile"
	at java.util.concurrent.atomic.AtomicIntegerFieldUpdater$AtomicIntegerFieldUpdaterImpl.<init>(AtomicIntegerFieldUpdater.java:284)
	at java.util.concurrent.atomic.AtomicIntegerFieldUpdater.newUpdater(AtomicIntegerFieldUpdater.java:76)
	at com.test.Test.main(Test.java:9)
Caused by: java.lang.IllegalAccessException: Class com.test.Test can not access a member of class com.test.Candidate with modifiers "private volatile"
	at sun.reflect.Reflection.ensureMemberAccess(Reflection.java:110)
	at sun.reflect.misc.ReflectUtil.ensureMemberAccess(ReflectUtil.java:103)
	at java.util.concurrent.atomic.AtomicIntegerFieldUpdater$AtomicIntegerFieldUpdaterImpl.<init>(AtomicIntegerFieldUpdater.java:280)
	... 2 more
```

显然这是因为反射时访问权限不足导致的。

---

第二，为了确保变量的修改效果及时得到体现，字段必须是volatile类型的。现在，我们将原正确代码中score的volatile修饰去掉，即变为：

```
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

public class Test {

    public static void main(String[] args) throws InterruptedException {
        final AtomicIntegerFieldUpdater<Candidate> aifu = AtomicIntegerFieldUpdater.newUpdater(Candidate.class, "score");
        // 检验AtomicIntegerFieldUpdater是否正确
        final AtomicInteger check = new AtomicInteger(0);
        final Candidate candidate = new Candidate();
        Thread[] threadArray = new Thread[10000];
        for (int i = 0; i < threadArray.length; i++) {
            threadArray[i] = new Thread() {
                @Override
                public void run() {
                    if (Math.random() > 0.4) {    // 约有60%的选民会投该候选人
                        aifu.incrementAndGet(candidate);
                        check.incrementAndGet();
                    }
                }
            };
            threadArray[i].start();
            threadArray[i].join();
        }
        System.out.println("AtomicIntegerFieldUpdater=" + candidate.score);
        System.out.println("AtomicInteger=" + check.get());
    }
}

class Candidate {
    // 得分并非是线程安全的
    int score;
}
```

编译通过，输出：

```
Exception in thread "main" java.lang.IllegalArgumentException: Must be volatile type
	at java.util.concurrent.atomic.AtomicIntegerFieldUpdater$AtomicIntegerFieldUpdaterImpl.<init>(AtomicIntegerFieldUpdater.java:292)
	at java.util.concurrent.atomic.AtomicIntegerFieldUpdater.newUpdater(AtomicIntegerFieldUpdater.java:76)
	at com.test.Test.main(Test.java:9)
```

错误很明显，就是由volatile导致的。

从严格意义上讲，这个规定其实是破坏了开闭原则的。因为我们已经无法完全做到在不修改原程序代码的基础上进行字段的线程安全化了。不过好在volatile通常不会对程序逻辑造成什么影响，因此通常这是可以接收的。

---

第三，同其他的Atomic一族的类一样，Updater底层是以Unsafe类来实现CAS操作的。具体来说，它调用得是Unsafe的objectFieldOffset()，而该方法并不支持类变量。因此，Updater自然也不会支持类变量的修改了。和前文的代码一样，我们将score设置为static的：

```
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

public class Test {

    public static void main(String[] args) throws InterruptedException {
        final AtomicIntegerFieldUpdater<Candidate> aifu = AtomicIntegerFieldUpdater.newUpdater(Candidate.class, "score");
        // 检验AtomicIntegerFieldUpdater是否正确
        final AtomicInteger check = new AtomicInteger(0);
        final Candidate candidate = new Candidate();
        Thread[] threadArray = new Thread[10000];
        for (int i = 0; i < threadArray.length; i++) {
            threadArray[i] = new Thread() {
                @Override
                public void run() {
                    if (Math.random() > 0.4) {    // 约有60%的选民会投该候选人
                        aifu.incrementAndGet(candidate);
                        check.incrementAndGet();
                    }
                }
            };
            threadArray[i].start();
            threadArray[i].join();
        }
        System.out.println("AtomicIntegerFieldUpdater=" + candidate.score);
        System.out.println("AtomicInteger=" + check.get());
    }
}

class Candidate {
    // 得分并非是线程安全的
    volatile static int score;
}
```

编译通过，输出：

```
Exception in thread "main" java.lang.IllegalArgumentException
	at sun.misc.Unsafe.objectFieldOffset(Native Method)
	at java.util.concurrent.atomic.AtomicIntegerFieldUpdater$AtomicIntegerFieldUpdaterImpl.<init>(AtomicIntegerFieldUpdater.java:297)
	at java.util.concurrent.atomic.AtomicIntegerFieldUpdater.newUpdater(AtomicIntegerFieldUpdater.java:76)
	at com.test.Test.main(Test.java:9)
```

objectFieldOffset()方法报了异常。