---
title: Java 并发-ReentrantLock
date: 2017-10-03 14:21:36
tags: [Java,并发,锁,Lock,ReentrantLock,重入锁]
categories: Java 并发
---

```
java.util.concurrent.locks.ReentrantLock;

public class ReentrantLock implements Lock, java.io.Serializable
```

JDK1.5引入了显式的锁机制。该机制以Lock接口及其实现类为代表。

ReentrantLock使用显式锁机制实现了synchronized的所有功能，在此基础上又增加了新功能，因此ReentrantLock可以视为synchronized的功能增强版：synchronized所监视的监视器对象可以是任何对象，那么能不能特化出一种对象就是专门用来处理锁，只作为锁对象使用呢？这个锁对象就是ReentrantLock。

实际上，Lock接口比synchronized更接近于底层：synchronized在软件层面依赖JVM实现锁，而Lock在硬件层面依赖特殊的CPU指令(CAS操作)实现锁。

不妨设有ReentrantLock的示例lock对象。lock.lock()在位置1立起了一道门，门上有一把锁即为lock对象。lock对象有一把唯一与之配对的钥匙，存放在lock对象的专有空间中。当线程执行到此处时，它会被这道门挡住，此时若钥匙还在专有空间中，它就可以拿起这把钥匙打开锁向下进行，因为唯一的一把钥匙在这个线程身上，那么后续所有被这把锁挡住的线程都无法进行(不仅仅是被位置1挡住的线程，若该锁也在位置2立起了门，那么被挡在位置2的线程也进行不下去，因为钥匙只有一把)。当持有钥匙的线程走到位置1对应的lock.unlock()处时，它将把钥匙还回lock对象的专有空间。

如果按照这个思路，竞争的其实不是锁，因为只要被门挡了，不论线程是否乐意，锁就在它们面前。真正竞争的其实是唯一配对的那一把钥匙(仅仅是用于理解，日常描述时我们还是会说线程持有了锁而非钥匙)。正如使用synchronized关键字时线程竞争的是监视器对象唯一的认可。

线程在等锁池等待的实现原理是阻塞原语线程挂起park()及线程恢复unpark()。详见[Java 并发-线程阻塞工具类LockSupport](/2017/10/07/Java 并发-线程阻塞工具类LockSupport/)。

<!-- more -->

简单的小例子

```
import java.util.concurrent.locks.ReentrantLock;

public class Test extends Thread {

    private static ReentrantLock LOCK = new ReentrantLock();

    private static int COUNT;

    @Override
    public void run() {
        for (int i = 0; i < 100000; i++) {
            Test.LOCK.lock();
            Test.COUNT++;
            Test.LOCK.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Test t1 = new Test();
        Test t2 = new Test();
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(Test.COUNT);
    }
}
```

输出：

```
200000
```

# 判断锁(的钥匙)是否被当前线程持有

```
import java.util.concurrent.locks.ReentrantLock;

public class Test {

    public static void main(String[] args) {
        ReentrantLock lock = new ReentrantLock();
        System.out.println("加锁前：" + lock.isHeldByCurrentThread());
        lock.lock();
        System.out.println("已获得锁：" + lock.isHeldByCurrentThread());
        lock.unlock();
        System.out.println("释放锁后：" + lock.isHeldByCurrentThread());
    }
}
```

输出：

```
加锁前：false
已获得锁：true
释放锁后：false
```

# 中断响应

使用synchronized时，如果一个线程在等待监视器对象的认可，那么等待的结果只有两种：要么获得认可结束等待；要么无限期的等下去。换句话说，等待监视器对象认可的线程是没有中断机制的。

ReentrantLock在加锁时可使用lockInterruptibly()使得所加的锁可响应中断：

```
import java.util.concurrent.locks.ReentrantLock;

public class Test extends Thread {

    private static ReentrantLock LOCK = new ReentrantLock();

    public Test(String name) {
        super(name);
    }

    @Override
    public void run() {
        try {
            Test.LOCK.lockInterruptibly();
            while(true) ;
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName() + "在等锁时被中断");
        }
        System.out.println(Thread.currentThread().getName() + "执行结束");
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Test("t1");
        Thread t2 = new Test("t2");
        t1.start();
        Thread.sleep(1000);
        t2.start();
        Thread.sleep(1000);
        t2.interrupt();
    }
}
```

执行后输出如下。且程序不会结束：因为t1不会停止。

```
t2在等锁时被中断
t2执行结束
```

注意响应中断的是等锁的这一行为，即卡在Test.LOCK.lockInterruptibly();这一行时发生了中断，卡在这一行的线程并没有获得锁。

# 限时等待

为了避免在申请不到锁时无限等待，上文中响应中断是其中一种方法，另一种方法是设置限时等待锁。

最为暴躁的等待方式为不等，即只要没申请到锁，立刻判为失败放弃申请，即：

```
import java.util.concurrent.locks.ReentrantLock;

public class Test extends Thread {

    private static ReentrantLock LOCK = new ReentrantLock();

    public Test(String name) {
        super(name);
    }

    @Override
    public void run() {
        if (Test.LOCK.tryLock()) {
            System.out.println(System.currentTimeMillis() + " -- " + Thread.currentThread().getName() + "获得锁");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LOCK.unlock();
            System.out.println(System.currentTimeMillis() + " -- " + Thread.currentThread().getName() + "正常执行结束");
        } else {
            System.out.println(System.currentTimeMillis() + " -- " + Thread.currentThread().getName() + "无法获得锁，执行结束");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Test("t1");
        Thread t2 = new Test("t2");
        t1.start();
        Thread.sleep(1000);
        t2.start();
    }
}
```

输出：

```
1507366890797 -- t1获得锁
1507366891815 -- t2无法获得锁，执行结束
1507366893817 -- t1正常执行结束
```

不那么粗暴的方式是设置一个等待时间：

```
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class Test extends Thread {

    private static ReentrantLock LOCK = new ReentrantLock();

    public Test(String name) {
        super(name);
    }

    @Override
    public void run() {
        try {
            if (Test.LOCK.tryLock(5, TimeUnit.SECONDS)) {
                System.out.println(System.currentTimeMillis() + " -- " + Thread.currentThread().getName() + "获得锁");
                Thread.sleep(3000);
                LOCK.unlock();
                System.out.println(System.currentTimeMillis() + " -- " + Thread.currentThread().getName() + "正常执行结束");
            } else {
                System.out.println(System.currentTimeMillis() + " -- " + Thread.currentThread().getName() + "无法获得锁，执行结束");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Test("t1");
        Thread t2 = new Test("t2");
        t1.start();
        Thread.sleep(1000);
        t2.start();
    }
}
```

输出：

```
1507367395772 -- t1获得锁
1507367398777 -- t1正常执行结束
1507367398777 -- t2获得锁
1507367401784 -- t2正常执行结束
```

# 公平锁

synchronized及普通的ReentrantLock是非公平的，即当许可或钥匙空出后，哪一个线程获得它是随机的。线程优先级只能影响获得概率的大小，无法确保先进入等待队列的线程就一定能先获得资源。因此可能产生饥饿的现象。

公平锁严格遵守FIFO原则，即不会因插队现象导致饥饿的发生(已获得资源的线程不愿意释放而导致的饥饿就没办法了)。

公平锁的构建方式：new ReentrantLock(true)。默认的new ReentrantLock()相当于传入了false，即默认为非公平锁。之所以这么设计是因为公平锁为了实现功能必然需要维护一个有序队列，如果不需要FIFO，则这是不必要的开销。

如下代码实现了公平锁，输出基本为t1,t2交替进行。如果改为new ReentrantLock();即使用非公平锁，因为锁机制的优化，线程会更容易获得其曾经持有过的锁，此时的输出为大块的连续t1及大块的连续t2交替进行。

```
import java.util.concurrent.locks.ReentrantLock;

public class Test extends Thread {

    private static ReentrantLock LOCK = new ReentrantLock(true);

    public Test(String name) {
        super(name);
    }

    @Override
    public void run() {
        while (true) {
            Test.LOCK.lock();
            System.out.println(Thread.currentThread().getName() + "获得了锁");
            Test.LOCK.unlock();
        }
    }

    public static void main(String[] args) {
        new Test("线程1").start();
        new Test("线程2").start();
    }
}
```