---
title: Java 并发-线程状态转换
date: 2017-07-12 17:40:36
tags: [Java,并发]
categories: Java 并发
---

# 进程与线程

进程(Process)是具有一定独立功能的程序在某个数据集合上的一次运行活动，是操作系统进行资源分配和调度的基本单位。

线程可以看作是轻量级的进程，是CPU调度和分派,比进程更小的能独立运行的基本单位。

进程在执行时通常拥有独立的内存单元，而线程之间可以共享内存。

简单来说，可以认为进程是线程的容器。举一个生活中的小例子：一家3口生活在他们的屋子里。在这里屋子就可以看作是进程。而在其中生活的一家3口就是线程。屋子为一家3口提供了电视，厨房，厕所等人类生活所需要的资源。因为有些资源是有限的，所以有时会产生冲突，例如电视只有一个，当孩子想看动画片时父亲就看不了体育比赛了。当然一个健康的家庭更多的时候还是在体现着协作(总冲突这个家也算是完了)，例如妈妈在厨房做饭，爸爸在书房工作，孩子在客厅玩耍，这样才能保证这个家庭平稳的生活下去。

<!-- more -->

# Java线程状态转换

![0.jpg](/images/blog_pic/Java 并发/线程状态转换/0.jpg)

图中的6个状态定义在Thread类的State枚举中：

```
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
```

Java语言定义了6种线程状态：

- 新建(NEW)：创建但尚未运行。即已调用new生成了线程对象，但是尚未调用start()启动。

- 可运行(RUNNABLE)：RUNNABLE状态的线程一定已获得了除CPU以外运行所需的所有资源，依是否正获得着CPU又可细分为两类：就绪(READY)及运行(RUNNING)(分别对应操作系统线程状态中的READY及RUNNING)。READY表明目前没有正在占用CPU，但是线程已经准备好了，正所谓万事俱备，只欠CPU，只要CPU为其分配了执行时间立即就可转为RUNNING状态运行。RUNNING表明线程正在占用着CPU。

- 无限期等待(WAITING)：该状态下的线程会让出CPU，已占用的锁或监视器对象的认可。

- 限期等待(TIMED_WAITING)：该状态下的线程会让出CPU，已占用的锁或监视器对象的认可。

- 阻塞(BLOCKED)：无限期等待(WAITING)及限期等待(TIMED_WAITING)代表了线程的意志，即线程本身想停下来等待，也就是所谓的"不想做"。而阻塞(BLOCKED)代表了JVM的意志，属于强制力，无论线程作何打算，JVM目前已没有足够的资源让线程运行下去，也就是所谓的"做不到"。

- 结束(TERMINATED)：线程已终止。

任意时间点一个线程有且仅有其中的一种状态。不同状态间的转换条件为：

新建(NEW) --> 可运行(RUNNABLE):start()

就绪(READY) --> 运行(RUNNING):CPU调度为线程分配CPU时间。

运行(RUNNING) --> 就绪(READY):CPU调度剥夺线程已得到的CPU，或线程主动调用yield()让出CPU。

可运行(RUNNABLE) --> 结束(TERMINATED):run()执行结束。

可运行(RUNNABLE) --> 阻塞(BLOCKED):申请资源(例如锁，监视器对象的认可，i/o流)而不得。

阻塞(BLOCKED) --> 可运行(RUNNABLE):之前申请不到的资源申请到了。

可运行(RUNNABLE) --> 限期等待(TIMED_WAITING):

- Thread.sleep()。

- 有TimeOut参数的Object.wait()。

- 有TimeOut参数的Thread.join()。

- LockSupport.parkNanos()。

- LockSupport.parkUntil()。

限期等待(TIMED_WAITING) --> 可运行(RUNNABLE):接到唤醒信号，或设定的限期到来时自动转换。如果需要，在苏醒后成功获得所需的锁或监视器对象的认可。

限期等待(TIMED_WAITING) --> 阻塞(BLOCKED):接到唤醒信号，或设定的限期到来时自动转换。在苏醒后未成功获得所需的锁或监视器对象的认可。

可运行(RUNNABLE) --> 无限期等待(WAITING):

- 没有TimeOut参数的Object.wait()。

- 没有TimeOut参数的Thread.join()。

- LockSupport.park()。

无限期等待(WAITING) --> 可运行(RUNNABLE):接到唤醒信号。如果需要，在苏醒后成功获得所需的锁或监视器对象的认可。

无限期等待(WAITING) --> 阻塞(BLOCKED):接到唤醒信号。在苏醒后未成功获得所需的锁或监视器对象的认可。

# sleep(long millis)

```
public static native void sleep(long millis) throws InterruptedException;
```

sleep(long millis)会抛出受检查异常InterruptedException，会让出已占有的CPU时间，会让给哪个线程与线程的优先级无关。

sleep(long millis)和锁及监视器对象的许可无关。

处于可运行(RUNNABLE)状态的线程调用sleep(long millis)后进入限期等待(TIMED_WAITING)状态。sleep(long millis)期限到来后恢复为可运行(RUNNABLE)状态。

# wait()/wait(long timeout)

wait()内部实际上就是调用wait(long timeout)，0表示无限期等待：

```
public final void wait() throws InterruptedException {
    wait(0);
}

public final native void wait(long timeout) throws InterruptedException;
```

wait()/wait(long timeout) 会抛出受检查异常InterruptedException，会让出已占有的CPU时间。

wait()/wait(long timeout) 必须配合synchronized关键字使用，并会放弃已得到的监视器对象的许可。

处于可运行(RUNNABLE)状态的线程调用wait()后进入无限期等待(WAITING)状态。被唤醒后将重新尝试获取监视器对象的认可，获取成功则恢复为可运行(RUNNABLE)状态，获取失败则进入阻塞(BLOCKED)状态。

处于可运行(RUNNABLE)状态的线程调用wait(long timeout)后进入限期等待(TIMED_WAITING)状态。被唤醒或限期到来后将重新尝试获取监视器对象的认可，获取成功则恢复为可运行(RUNNABLE)状态，获取失败则进入阻塞(BLOCKED)状态。

# yield()

```
public static native void yield();
```

yield()只会将CPU让给相同优先级或更高优先级的线程。

处于运行(RUNNING)状态的线程调用yield()后进入就绪(READY)状态。

# 等锁池与等待池

等锁池及等待池均属于对象。等锁池(lock pool)是指等待synchronized及Lock的线程所处的队列。等待池(wait pool)是指调用 wait()/wait(long timeout) 后线程所处的队列。