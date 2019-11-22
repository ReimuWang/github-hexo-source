---
title: Java 并发-线程阻塞工具类LockSupport
date: 2017-10-07 17:51:36
tags: [Java,并发,LockSupport,阻塞]
categories: Java 并发
---

[Java 并发-Thread类](/2017/10/05/Java 并发-Thread类/)中介绍过一对挂起-恢复线程的方法，它们是Thread类的suspend()及resume()方法。因为线程在使用suspend()方法挂起时不会让出任何资源，因此这一对方法现在已被标记为废弃。

那么该如何做到线程的挂起-恢复呢？Object类的wait()及notify()虽然也能起到挂起-恢复的功能，但它们最本质的功能是为synchronized机制提供通信，即便做到了线程的挂起-恢复也仅仅只是附带的结果，而非手段。而且wait()及notify()必须在synchronized代码块的范围内由其所从属的监视器对象调用，不仅灵活度不够，也不合理：线程的挂起及恢复是线程自身意愿的体现，不应该必须限制在synchronized同步代码块内，因为事实上二者并没有必然的联系。

<!-- more -->

LockSupport就是Java API针对这个问题提供的解决策略。它是一个方便使用的线程阻塞工具，可以看作是suspend()及resume()的替代品，它同样可以在线程内任意位置让线程挂起。和suspend()及resume()相比，它还有如下改进：

- 被挂起的线程将让出一切资源，即处于WAITING状态。

- 不会出现类似于resume()被先调用导致唤醒信号丢失的情况。

和wait()及notify()相比，LockSupport的不同之处在于：

- 不会抛出InterruptedException。

- 和synchronized机制无关。不需局限在synchronized代码块的范围内。

可以这样认为，在线程的挂起-恢复这个问题上，LockSupport吸收了suspend()/resume()及wait()/notify()的优点。

java.util.concurrent.locks.LockSupport的类定义为：

```
public class LockSupport
```

常用方法如下：

```
/**
 * 挂起当前线程
 */
public static void park()

public static void parkNanos(long nanos)

public static void parkUntil(long deadline)

/**
 * 恢复传入线程
 */
public static void unpark(Thread thread)
```

下面来看一个小例子：

```
import java.util.concurrent.locks.LockSupport;

public class Test {

    public static void main(String[] args) {
        Runnable  r = new Runnable() {
            @Override
            public void run() {
                LockSupport.park();
                System.out.println("线程恢复并结束");
            }
        };
        Thread t = new Thread(r);
        t.start();
        LockSupport.unpark(t);
    }
}
```

输出：

```
线程恢复并结束
```

关于这个小例子，需要注意的是：事实上我们无法保证unpark()一定会在park()之前被调用。对于suspend()/resume()而言，这可能会导致恢复消息的丢失。然而对于LockSupport而言，无论unpark()与park()的调用顺序为何，都能保证恢复消息生效。其原因就在于LockSupport使用了类似于信号量的机制，每个线程都有一个唯一的信号量，可以称之为许可。unpark()会将该许可置为可用，park()会检查这个许可，若不可用则挂起并等待，若可用则消费这个许可并继续进行。

park()会将线程转为WAITING状态，如果我们将此时的线程dump出来，还会发现这个WAITING状态被贴心的标记为是由park()引起的。如果我们想在dump时获得更多的信息，还可使用如下方法：

```
public static void park(Object blocker)
```

blocker被称为被挂起线程的阻塞对象，这样在分析问题时，就能更加的方便。

前文在比较LockSupport与wait()/notify()的差异时，提到LockSupport不会抛出InterruptedException，这并不意味着park()不响应中断，它只是不抛出InterruptedException：它会什么都不做，默默的返回。例子如下：

```
import java.util.concurrent.locks.LockSupport;

public class Test {

    public static void main(String[] args) {
        Runnable  r = new Runnable() {
            @Override
            public void run() {
                LockSupport.park();
                System.out.println("线程继续，中断标志=" + Thread.interrupted());
            }
        };
        Thread t = new Thread(r);
        t.start();
        t.interrupt();
    }
}
```

输出：

```
线程继续，中断标志=true
```