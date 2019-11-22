---
title: Java 并发-Thread类
date: 2017-10-05 11:25:36
tags: [Java,并发,线程,Thread]
categories: Java 并发
---

```
java.lang.Thread

public class Thread implements Runnable
```

<!-- more -->

# 线程终止(stop)

Thread类提供了终止线程的方法：

```
public final void stop()
```

类似于linux中的kill -9对进程的处理，调用该方法后，会由JVM强制从外部粗暴的杀掉线程。被杀掉的线程没有机会对其已做的事情做妥善的处理，可能会导致数据不一致的问题(已经改到一半的数据没机会回滚)。因此stop()是一个被废弃的方法。

用户程序可自己实现安全停止线程的方法：

```
public class Test extends Thread {

    private volatile boolean ifStop;

    public void stopMe() {
        this.ifStop = true;
    }

    @Override
    public void run() {
        while (true) {
            if (this.ifStop) break;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Test test = new Test();
        test.start();
        Thread.sleep(3000);
        test.stopMe();    // 经过3s的睡眠后，线程终止
    }
}
```

# 线程中断(interrupt)

stop()方法之所以只能强制停止线程是因为JVM无法了解用户程序的业务逻辑，即无法判断何时才算是"告一段落"，何时终止线程才不会写坏数据。而上文中实现的安全停止线程的方法其实思路很简单，也很机械：在合适的，不会写坏数据的位置设置一个停止的标记，然后在合适的时机激活它。除了自己实现外，JDK也提供了一套中断机制来实现这个功能。

Thread类中和中断有关的方法有3个：

```
public void interrupt()
```

和上文的stopMe()类似，仅仅将中断标记设置为true，如果没有具体的中断处理逻辑的话单纯调用该方法是没有意义的。

```
public boolean isInterrupted()
```

实例方法：通过中断标记判断线程是否已被标记为中断。

```
public static boolean interrupted()
```

类方法：通过当前线程的中断标记判断当前线程是否已被中断。

如果使用Java的中断机制，上文中实现的安全停止线程的方法变为：

```
public class Test extends Thread {

    @Override
    public void run() {
        while (true) {
            if (this.isInterrupted()) break;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Test test = new Test();
        test.start();
        Thread.sleep(3000);
        test.interrupt();    // 经过3s的睡眠后，线程终止
    }
}
```

或：

```
public class Test extends Thread {

    @Override
    public void run() {
        while (true) {
            if (Thread.interrupted()) break;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Test test = new Test();
        test.start();
        Thread.sleep(3000);
        test.interrupt();    // 经过3s的睡眠后，线程终止
    }
}
```

除了减少代码量外，因为中断机制是JDK所实现的基本功能，自然也能配合JDK的其他功能。sleep(), wait()/wait(long millis), join()/join(long millis) 等均可以响应中断。

sleep()响应中断的例子：

```
public class Test extends Thread {

    @Override
    public void run() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            System.out.println("线程被中断");
            System.out.println("此时的中断标志为：" + this.isInterrupted());
        }
    }

    public static void main(String[] args) {
        Test test = new Test();
        test.start();
        test.interrupt();
    }
}
```

执行后的输出为：

```
线程被中断
此时的中断标志为：false
```

这段代码的结果说明两点问题：

1. 处于睡眠状态的线程在接到中断信息后会以抛出受检查异常InterruptedException的方式响应中断(这也是为什么sleep()要强制检查InterruptedException的原因)。

2. sleep()响应过中断后，线程的中断标记被重新置为false。可以理解为一次中断请求仅会被处理一次。

wait()响应中断的例子：

```
public class Test extends Thread {

    @Override
    public void run() {
        synchronized (Test.class) {
            try {
                Test.class.wait();
            } catch (InterruptedException e) {
                System.out.println("线程被中断");
                System.out.println("此时的中断标志为：" + this.isInterrupted());
            }
        }
    }

    public static void main(String[] args) {
        Test test = new Test();
        test.start();
        test.interrupt();
    }
}
```

执行后的输出为：

```
线程被中断
此时的中断标志为：false
```

对该代码的分析同sleep()的例子。

join()响应中断的例子：

```
public class Test implements Runnable {

    private Thread t;

    public Test(Thread t) {
        this.t = t;
    }

    @Override
    public void run() {
        try {
            this.t.join();
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName() + "被中断");
            this.t.interrupt();
        }
    }

    public static void main(String[] args) {
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                try {
                    synchronized (Test.class) {
                        Test.class.wait();
                    }
                } catch (InterruptedException e) {
                    System.out.println(Thread.currentThread().getName() + "被中断");
                }
            }
        };
        Thread t1 = new Thread(r1, "t1");
        Thread t2 = new Thread(new Test(t1), "t2");
        t1.start();
        t2.start();
        t2.interrupt();
    }
}
```

执行后程序会在极短的时间内结束，且输出如下：

```
t2被中断
t1被中断
```

对该代码的分析同wait()的例子。因为调用t.join()实质上就是让当前线程wait在了t的等待队列上。

该代码首先t2.interrupt();触发了this.t.join();的中断，因为其实质上是将当前线程(即t2)加入到了t1的等待队列中，因此响应中断的实质是t1的wait()。在this.t.join();的中断处理逻辑中又触发了t1的中断，此时t1在Test.class的等待队列中等待，触发中断后t1也进入中断响应逻辑处理代码，程序得以结束。

# 挂起(suspend)与继续执行(resume)

Thread提供了如下方法：

```
public final void suspend()

public final void resume()
```

挂起(suspend)与继续执行(resume)所实现的功能基本与Object类的wait()及notify()相同。不同之处主要有两点：

1. wait()及notify()必须与synchronized配合使用。suspend()及resume()则没有限制。

2. 处于可运行(RUNNABLE)状态的线程调用wait()后进入无限期等待(WAITING)状态，并放弃CPU和已获得的监视器对象的认可；处于可运行(RUNNABLE)状态的线程调用suspend()后依然还是可运行(RUNNABLE)状态，且不会放弃任何已获得的资源；也就是说，suspend()纯粹是线程内部的打算，和用户程序自己设计实现的忙等待并发程序很类似：在JVM看来，进入挂起状态的程序依然是在运行的，和非挂起状态没什么不同。正因为如此，挂起(suspend)与继续执行(resume)被标记为了废弃方法。

# 等待线程结束(join)

Thread提供了如下方法：

```
public final void join() throws InterruptedException {
    join(0);
}

public final synchronized void join(long millis) throws InterruptedException
```

t.join()相当于让当前线程一直等到t执行结束后再继续向下执行。

join()本质上是由join(long millis)实现的，而join(long millis)本质上是由wait(long millis)。换句话说t.join()的本质为：将当前线程加入t的等待队列，再直白些说，就是调用了t.wait()，在t执行结束后再调用t.notifyAll()唤醒所有线程。这种逻辑带来了两个比较糟糕的问题：

问题1：notifyAll()唤醒的是所有挂在t等待队列上的线程，也就是说不是因为join()被挂上去的线程也被唤醒了，代码求证如下：

```
public class Test {

    public static void main(String[] args) {
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        Thread t1 = new Thread(r1);
        t1.start();
        synchronized (t1) {
            try {
                t1.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("after wait on t1");
    }
}
```

设定的1s过去后，输出了after wait on t1。而如果注掉t1.start();，程序将不会有输出，且main线程会因等待t1而无法完成。其原因就在于t1为了支持join()方法，会在结束时调用t1.notifyAll()方法，此时本不希望被唤醒的main方法线程就被意外唤醒了。

问题2：如果我们主动调用了notifyAll()唤醒了join()在t上的线程，甚至于我们足够不幸，调用notify()时随机唤醒了join()在t上的线程，这都不是我们所希望的(仅是理论分析，代码求证失败了，可能是还有其他保护机制)。

为了避免这两种情况的出现，不要将Thread的实例对象作为监视器对象。

# 谦让(yield)

Thread类提供了如下方法：

```
public static native void yield();
```

运行(RUNNING)状态的当前线程调用yield()后会进入就绪(READY)状态，即主动让出CPU。需要注意的有两点：

1. 无论运行(RUNNING)抑或就绪(READY)，都是可运行(RUNNABLE)。调用yield()后只是让出了已占有的CPU资源，并没有让出其他资源。

2. 调用yield()进入就绪(READY)状态的线程依然还会依据基本法进行下一轮的资源竞争。

3. CPU只会被让给具有相同或更高优先级的线程。

4. 无需运行于synchronized代码块中。