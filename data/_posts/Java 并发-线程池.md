---
title: Java 并发-线程池
date: 2017-10-03 14:33:36
tags: [Java,并发,线程池]
categories: Java 并发
---

# 线程池的产生背景

在使用线程时，我们常会遇到以下的业务需求：以runnable1为核心声明thread1。不久后，我们又需要runnable1的功能了，但是此时thread1尚未执行完，或已因run()的结束被JVM回收，总之是没办法再利用了，我们只能以runnable1为核心声明一个新的线程thread2...类比下去，我们还可能需要生成thread3，thread4，thread5...

乍看下来似乎没毛病，然而，如果线程执行完成的速度跟不上线程创建的速度，程序所使用的线程数就会不断上升，但更底层操作系统所能承载的线程总数一定是有限的，持续下去，必然会导致没有充分的资源分配新的线程，使得程序崩溃。

即便线程执行完成的速度跟得上线程创建的速度，依然还会存在问题，只是没那么糟糕了而已：虽然与进程相比，线程是一种轻量级的工具，但其创建与回收依然需要花费时间与资源，而这其实是没有必要的。在我们所模拟的场景中，真正重要的只有runnable1，至于thread1，thread2...不过是为了完成runnable1功能的载体，事实上，如果我们能做到thread1在run()结束后不被回收，同时也能容忍只有一个线程串行执行的效率的话，那么只用thread1串行执行完全可以完成功能。

我们当然不能接受串行执行(否则引入线程的意义何在？)，不过照顾底层操作系统的承受能力，为runnable1设置一个允许创建的最大线程数还是很合理的，比如我们规定最大线程数是2，那么可能会有如下执行流程：

- 需求1:需要runnable1的功能，创建thread1。

- 需求2:需要runnable1的功能，创建thread2。

- 需求3:需要runnable1的功能，但是能创建的线程数已达上限。且thread1及thread2仍未执行完，只能继续等待。

- 需求2率先执行完了，这样thread2便空了出来，可以用来执行需求3。

这便是线程池的雏形了，它是系统性能与程序执行效率之间制衡的结果。使用线程池后，创建线程变为向线程池讨要线程，销毁线程变为向线程池归还线程。至于线程创建，销毁的真正细节都交由线程池管理。这与数据库连接池等连接池的理念是相同的。

<!-- more -->

# Java API中线程池的层次结构

Java API提供了完善的线程池组件，其顶层为接口Executor，它的代码很简单，全部代码如下：

```
package java.util.concurrent;

public interface Executor {

    void execute(Runnable command);
}
```

这便是我们在上文讨论的线程池所欲实现的核心内容了：使用者向线程池提交核心功能command，它自行创建(或者是复用，总之与使用者无关)线程实现对应的功能。

它的子接口为ExecutorService，全部代码如下：

```
package java.util.concurrent;

import java.util.List;
import java.util.Collection;
import java.security.PrivilegedAction;
import java.security.PrivilegedExceptionAction;

public interface ExecutorService extends Executor {

    /**
     * 关闭线程池
     * 注意这只是不再接受新的请求了，已在执行的线程会执行完成
     */
    void shutdown();

    List<Runnable> shutdownNow();

    /**
     * 线程池已关闭(已调用过shutdown()或shutdownNow())返回true，反之返回false
     */
    boolean isShutdown();


    /**
     * 若未调用过shutdown()或shutdownNow()则返回false
     * 若已调用过shutdown()或shutdownNow()，则所有线程执行完成返回true，反之返回false
     */
    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

execute，submit，invoke均是用来执行线程的，其不同之处主要在于是否能得到线程执行后的返回值Future，可详见[Java 并发-Future](/2018/01/09/Java 并发-Future/)。简单来说，如果我们不需要收集线程的执行结果，则使用execute就好。

ExecutorService有一个子接口ScheduledExecutorService，它在ExecutorService的基础上扩展了在给定时间执行某任务的功能：如在某个固定的延时之后执行，或者周期性的执行某个任务。

实现ExecutorService接口骨架功能的类为java.util.concurrent.AbstractExecutorService，其类定义如下：

```
public abstract class AbstractExecutorService implements ExecutorService
```

这是一个抽象类，其最常用的非抽象子类为java.util.concurrent.ThreadPoolExecutor，其类定义如下：

```
public class ThreadPoolExecutor extends AbstractExecutorService
```

这便是我们最常用的线程池的实现了。不过Java API为了简化程序员的操作，还贴心的为我们提供了线程池的生产工厂java.util.concurrent.Executors，它的类定义如下：

```
public class Executors
```

该工厂能生产我们最需要的那几种线程池，例如：

```
public static ExecutorService newFixedThreadPool(int nThreads)

public static ExecutorService newSingleThreadExecutor()

public static ExecutorService newCachedThreadPool()

public static ScheduledExecutorService newSingleThreadScheduledExecutor()

public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
```

上文所述的接口与类均位于java.util.concurrent包中，我们可以用一张图来描述这些类/接口之间的关系：

![0.jpg](/images/blog_pic/Java 并发/线程池/0.jpg)

其中，实线是继承，虚线是实现接口。

# Executors提供的常用线程池

下面我们来依次详细讨论下Executors所提供的最常用的那些线程池。

---

```
public static ExecutorService newFixedThreadPool(int nThreads)
```

返回一个固定线程数量的线程池。当有一个新的任务被提交时，若此时已没有空闲线程，则该任务会被暂存于一个等待队列中，待有线程空闲后再处理该任务。

---

```
public static ExecutorService newSingleThreadExecutor()
```

返回一个只有一个线程的线程池，可类比理解为newFixedThreadPool中nThreads=1的情况(只是帮助理解，实际还是有区别的)。

---

```
public static ExecutorService newCachedThreadPool()
```

返回一个根据实际情况调整线程数量的线程池：当有一个新的任务被提交时，若此时已没有空闲线程，则会创建新线程。创建的线程在执行完成后默认再保存60秒，若在此期间有复用请求则复用该线程，反之则在到时间后销毁。

---

```
public static ScheduledExecutorService newSingleThreadScheduledExecutor()
```

返回一个只有一个线程的，有定时任务功能的线程池。

---

```
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
```

返回一个可指定线程数量的，有定时任务功能的线程池。

---

下面我们以newFixedThreadPool为例，给出一个小例子：

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {

    private static Long BEGIN = System.currentTimeMillis();

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(5);
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]" + Thread.currentThread().getId() + " run");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        for (int i = 0; i < 10; i++) es.execute(r);
        es.shutdown();
    }
}
```

输出：

```
[5]12 run
[5]11 run
[6]9 run
[7]13 run
[7]10 run
[1005]11 run
[1005]12 run
[1006]9 run
[1007]13 run
[1007]10 run
```

程序被分为两组，两组间的间隔约为1秒。且第二组其实并未创建新线程，只是对第一组线程的复用。

我们不妨将上例的线程池换为newCachedThreadPool，其余不做改动：

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {

    private static Long BEGIN = System.currentTimeMillis();

    public static void main(String[] args) {
        ExecutorService es = Executors.newCachedThreadPool();
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]" + Thread.currentThread().getId() + " run");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        for (int i = 0; i < 10; i++) es.execute(r);
        es.shutdown();
    }
}
```

输出：

```
[5]9 run
[6]13 run
[5]10 run
[5]11 run
[6]14 run
[6]15 run
[6]12 run
[6]18 run
[6]17 run
[6]16 run
```

几乎在同时创建了10个新线程。

通览Executors工厂提供的线程池，可分为两大类：一类是上文给出过小例子的，诸如newFixedThreadPool，newCachedThreadPool等普通的线程池，这类线程池会在有空闲线程后立即执行。另一类就是方法名中包含关键字Scheduled，返回ScheduledExecutorService的定时任务线程池，这类线程池所实现的功能类似于linux中的crontab命令。

ScheduledExecutorService的全部代码如下：

```
package java.util.concurrent;
import java.util.concurrent.atomic.*;
import java.util.*;

public interface ScheduledExecutorService extends ExecutorService {

    /**
     * 在延迟时间delay之后，对任务进行一次调度
     */
    public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit);

    /**
     * 在延迟时间delay之后，对任务进行一次调度
     */
    public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                           long delay, TimeUnit unit);

    /**
     * 创建一个周期性任务
     * 任务开始于给定的初始延时initialDelay之后
     * 后续的任务按照给定的周期进行：
     * 后续第一个任务于initialDelay+period执行
     * 后续第二个任务于initialDelay+2*period执行
     * 依此类推
     */
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);

    /**
     * 创建一个周期性任务
     * 任务开始于给定的初始延时initialDelay之后
     * 每当前一个任务完成后，再过延时delay后，开始下一个任务
     */
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);

}
```

下面给出一个scheduleAtFixedRate的小例子，例子中任务执行耗时1秒，初始延迟2秒，每隔3秒执行一次：

```
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class Test {

    private static Long BEGIN = System.currentTimeMillis();

    public static void main(String[] args) {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]" + Thread.currentThread().getId() + " run");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        ScheduledExecutorService ses = Executors.newScheduledThreadPool(10);
        ses.scheduleAtFixedRate(r, 2, 3, TimeUnit.SECONDS);
    }
}
```

截取最开始的那部分输出：

```
[2007]9 run
[5007]9 run
[8008]11 run
[11008]9 run
[14008]12 run
[17008]11 run
[20009]13 run
[23009]9 run
[26009]14 run
[29010]12 run
```

确实是按照预期执行了。不过需要注意的是，尽管其实只需1个线程就足够了，线程池仍然创建了新的其他线程(知道有这么个事就行了，具体的决策交给线程池)。

在完成这个小例子后，我们不禁会想：如果任务的执行时间大于循环周期的话，那么会出现任务堆叠的现象吗？我们可以将任务耗时调整为5秒，其余不变：

```
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class Test {

    private static Long BEGIN = System.currentTimeMillis();

    public static void main(String[] args) {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]" + Thread.currentThread().getId() + " run");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        ScheduledExecutorService ses = Executors.newScheduledThreadPool(10);
        ses.scheduleAtFixedRate(r, 2, 3, TimeUnit.SECONDS);
    }
}
```

截取最开始的那部分输出：

```
[2009]9 run
[7009]9 run
[12010]11 run
[17010]9 run
[22010]12 run
[27010]11 run
```

循环周期被自动调整为了5秒，并没有出现任务堆叠的情况。这样设计是很合理的：因为任务的执行时间大于循环周期的话，随着时间的推移，积压的任务会越来越多(能用于执行任务的线程毕竟是有限的)，无法执行的话还不如最开始就不要积压，降低执行频率。

最后，在使用定时任务，尤其是周期性循环的周期任务时，要尤为注意异常的捕获，因为只要一个任务抛出了未被捕获的异常，后续任务都会停止。

# Executors提供的常用线程池的内部实现

Executors工厂所提供的常用线程池本质上是Java API为我们创建的套路化的，常见功能的线程池，其内部还是需要创建ExecutorService的具体实现。以newFixedThreadPool，newSingleThreadExecutor，newCachedThreadPool这3个方法为例，他们均返回了ExecutorService的实现，源码为：

```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

原来如此，这3种线程池本质上都是调用同一个构造函数新建了一个ThreadPoolExecutor对象(newSingleThreadExecutor在外面又包了一层，不过影响不大)。也就是说它们3个本质上是一回事，只是生成的参数有所不同。它们所调用的ThreadPoolExecutor的构造函数为：

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

哦，原来只是一个壳子，它内部调用的那个构造函数为：

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

这次不是壳子了，不过所做的也无非就是给对象赋个值，没什么别的逻辑，确实很有构造函数的样子。我们需要重点关注的是它的入参。

先说相对简单的，threadFactory指线程工厂，用于创建线程。handler指拒绝策略，指明当任务超出线程池的承载能力时该如何应对。这两个参数也是上文那个壳子函数中少掉的两个参数，用的都是系统的默认值，并未由用户指定。

corePoolSize指预期中该线程池通常情况下会包含的线程数。当一个新的任务被提交至线程池，若有空闲线程则直接复用。在没有空闲线程的情况下，若此时已有线程数在[0,corePoolSize)之间，说明仍在预期的范围之内，可直接创建新线程。若已有线程数已达到corePoolSize，则说明超过了预期，新任务会被放入阻塞队列workQueue中，等待执行时机。

上文叙述的是在预期范围内的正常情况，而异常情况指得就是阻塞的任务数量超出了workQueue的承载极限。对于这种情况，ThreadPoolExecutor并没有直接判定失败进而调用handler的拒绝策略，而是又给出了一个弹性的空间：若此时已有线程数量在[corePoolSize,maximumPoolSize)的范围内，那么仍允许创建新线程。而若线程数已达maximumPoolSize而仍要创建新线程，则会判定失败并调用handler的拒绝策略。

不过，异常状况终究是异常状况，在(corePoolSize,maximumPoolSize]范围内的线程是紧急事态下的紧急应对，因此为它们设置了失效时间keepAliveTime(unit就是这个时间的单位)：一旦它们闲置的时间超过了这个失效时间，换句话说，危机应该是已经过去了，它们就会被销毁。

上文所述的调度逻辑可通过ThreadPoolExecutor中下述的核心调度代码得以体现：

```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();    // c代表当前线程池
    // workerCountOf(c)为当前线程池持有的线程总数
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))    // 创建新线程
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {    // 欲进入等待队列
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))    // 进入等待队列失败，将任务交还给线程池，判断是否达到maximumPoolSize
        reject(command);
}
```

在上文所述的ThreadPoolExecutor的入参中，还需特别展开论述的就是workQueue了，通常情况下，会传给ThreadPoolExecutor的阻塞队列有如下几种：

---

**ArrayBlockingQueue**

即有界任务队列。该队列的构造函数均包含一个容量参数，表示该队列所能承载的极限容量：

```
public ArrayBlockingQueue(int capacity)

public ArrayBlockingQueue(int capacity, boolean fair)

public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c)
```

---

**LinkedBlockingQueue**

即无界任务队列。除非系统资源耗尽，否则LinkedBlockingQueue可一直插入新元素。

---

**PriorityBlockingQueue**

即优先任务队列。它是一个特殊的无界任务队列，可以按照自身设定好的优先级计算规则控制任务执行的先后顺序。不过更确切的说，也可以认为无界任务队列是一种特殊的优先任务队列：它的优先计算规则为时间上的FIFO。

---

**SynchronousQueue**

即直接提交队列。其实这已经不能算作一种队列了：它并没有容量，即容量为0。换句话说，任何一个对SynchronousQueue的写需要等待一个对SynchronousQueue的读，反之亦然。因此，SynchronousQueue与其说是一个队列，更像是一个数据的中转站。

---

了解了线程池的生成方式后，我们终于可以逐一的分析上文列出的那3种线程池了：

---

```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

线程数在[0,nThreads)范围内时(nThreads即为设定好的容量)，当有新任务提交又没有空闲线程时(其实有空闲线程时也有可能创建新线程，一切看线程池的调度)，会创建新线程。当线程数已达nThreads，又没有空闲线程，此时新任务会被提交给阻塞队列LinkedBlockingQueue，这是一个无界任务队列，只要系统资源没有崩溃，理论上可以无限的提交下去。这样一来参数2，3，4即maximumPoolSize，keepAliveTime，unit实际上等同于无效了：因为不可能出现阻塞队列满了的情况。

使用LinkedBlockingQueue虽然不会导致线程池失败，然而在任务提交速度远大于消费速度的场合却有可能导致系统因资源耗尽而崩溃，需要谨慎使用。

---

```
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

不看外面包的那层FinalizableDelegatedExecutorService的话，完全可以按照nThreads=1的newFixedThreadPool来分析，就不再赘述了。

---

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

corePoolSize=0，这意味着该线程池稳定状态下的线程数是0，即所有线程都是临时工。新任务会被无条件的送入阻塞队列。而它的阻塞队列又是SynchronousQueue，所以立刻又会被踢皮球一样送回来，进入所谓的战时状态，最多可创建Integer.MAX_VALUE个线程。这个数字基本就可被视为无限大了(因为不会有系统能顶得住开这么多的线程的)。这些线程在闲置后均会被保持60秒，若无人使用则销毁。

在任务提交速度远大于消费速度的场合下，系统可能会因创建线程过多而卡死崩溃。这个问题比使用newFixedThreadPool时可能遇到的资源耗尽问题要更严峻一些：因为线程资源往往是更易于耗尽的。

---

# 线程池的拒绝策略

前文我们提到了线程池实现类ThreadPoolExecutor的构造函数：

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

其中有一个参数名为handler，代表着线程池的拒绝策略，在使用线程池工厂Executors创建常用线程池时并不需要关注这个参数，Executors会自动为我们补上默认的拒绝策略defaultHandler。

所谓的拒绝策略，就是当任务提交量超出了线程池的承载能力时需要做的补救策略。相当于是一种变相的try-catch。

拒绝策略的顶层接口为RejectedExecutionHandler，它的全部代码如下：

```
package java.util.concurrent;

public interface RejectedExecutionHandler {

    /**
     * 
     * @param r Runnable, 导致请求失败的那个任务
     * @param executor ThreadPoolExecutor, 当前线程池
     */
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

Java API提供了4种实现类：

---

java.util.concurrent.ThreadPoolExecutor$AbortPolicy

最为简单粗暴的拒绝策略：直接抛出异常，停止线程池。这也是Java API默认使用的策略。

我们可以追溯下源码：

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

其中的defaultHandler为ThreadPoolExecutor对象的字段：

```
private static final RejectedExecutionHandler defaultHandler =
    new AbortPolicy();
```

---

java.util.concurrent.ThreadPoolExecutor$CallerRunsPolicy

该策略会将无法完成的任务交还给调用者线程。这样做不会使线程池关闭，也不会丢弃任务，但却会使调用者线程的压力增大，也就是俗称的甩锅策略。

---

java.util.concurrent.ThreadPoolExecutor$DiscardOldestPolicy

该策略会丢弃最老的一个请求(也就是即将被执行的请求)。

---

java.util.concurrent.ThreadPoolExecutor$DiscardPolicy

该策略会丢弃无法处理的请求(也就是最新提交的那个请求)。

---

当然，我们也可以自己实现RejectedExecutionHandler：

```
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Test {

    private static long BEGIN = System.currentTimeMillis();

    public static void main(String[] args) {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]" + Thread.currentThread().getId() + " start...");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        RejectedExecutionHandler reh = new RejectedExecutionHandler() {
            @Override
            public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                System.out.println("[" + (System.currentTimeMillis() / Test.BEGIN) + "]" + r.toString() + " is discard");
            }
        };
        ExecutorService es = new ThreadPoolExecutor(5, 5, 0L, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(3), Executors.defaultThreadFactory(), reh);
        for (int i = 0; i < 10; i++) es.execute(r);
        es.shutdown();
    }
}
```

输出：

```
[3]10 start...
[1]com.test.Test$1@620a9239 is discard
[3]13 start...
[3]12 start...
[3]11 start...
[3]9 start...
[1]com.test.Test$1@620a9239 is discard
[103]11 start...
[106]10 start...
[105]12 start...
```

需要注意的一个点是reh的rejectedExecution()是在main方法线程中被执行的：因为线程池es就是main方法线程创建的。

# 线程创建工厂

我们继续分析线程池实现类ThreadPoolExecutor的构造函数：

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

上文已分析了handler，那么我们索性将其分析完，再来说一说另一个默认的参数：threadFactory。

在[Java 并发-线程组](/2017/10/06/Java 并发-线程组/)中，我在比较线程组与线程池的区别时曾说到：

**线程池管理的并非一个个已经成型的线程。即用以构建线程池中的线程所需提供的素材并非线程本身，而是生成线程的核心部件Runnable。也就是说线程池并不希望我们确定下来线程再交给它，这样它也就没法管理了。使用线程池时我们只管从池子里取线程即可，该线程是基于我们传入的Runnable生成的，因此可以满足我们的业务需求，至于这个线程是何时生成的，怎么生成的我们一概不管。而线程组是在明确的给线程打上编号：即某某线程属于某某组。一定要先有一个明确的线程才行。因此二者是无法协同工作的。**

结合本文，相信大家会有一个更深入的了解。

不过，虽然线程池不希望使用者看到它所持有的线程的细节，但为了执行任务，它总是要创建线程的，所用的就是下面要说的这个默认参数threadFactory。

ThreadFactory是一个接口，它的全部代码如下：

```
package java.util.concurrent;

public interface ThreadFactory {

    Thread newThread(Runnable r);
}
```

够简洁，我喜欢！我们可以先来看看ThreadPoolExecutor的默认实现是什么样子的，也就是Executors.defaultThreadFactory()方法：

```
public static ThreadFactory defaultThreadFactory() {
    return new DefaultThreadFactory();
}
```

依然灰常的直接，而这个DefaultThreadFactory全称为java.util.concurrent.Executors$DefaultThreadFactory。它的全部代码如下：

```
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

大概看下来，它的newThread()所创建的线程就是普通的线程，还特意确保了它的普通性：不是守护线程，优先级为NORM_PRIORITY。

理所当然的，我们也可以自己写ThreadFactory的实现类：

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Test {

    private static long BEGIN = System.currentTimeMillis();

    public static void main(String[] args) {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]" + Thread.currentThread().getName() + Thread.currentThread().getId() + " start...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        ThreadFactory tf = new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r, "tf制造");
                t.setDaemon(true);
                return t;
            }
        };
        ExecutorService es = new ThreadPoolExecutor(5, 5, 0L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(), tf);
        for (int i = 0; i < 10; i++) es.execute(r);
    }
}
```

输出：

```
[2]tf制造10 start...
[2]tf制造12 start...
[2]tf制造11 start...
[2]tf制造9 start...
[3]tf制造13 start...
```

这个输出就比较有意思了，因为线程均被设置为了守护线程，因此在main方法结束后正在sleep的那5个线程会被强制终止，线程池也被强制关闭，已提交在阻塞队列中的另5个任务自然也失去了执行的机会。

# 线程池的继承与扩展

讨论了这么多，不知大家是否会产生这样的疑问：线程池究竟是如何执行提交给它的任务的呢？

在ThreadPoolExecutor这个实现中，执行任务的工作交于了它的内部类Worker：

```
private final class Worker extends AbstractQueuedSynchronizer implements Runnable
```

它实现了Runnable接口。每个任务在执行时都会对应一个Worker实例。并通过该实例的run()方法执行：

```
public void run() {
    runWorker(this);
}
```

runWorker()方法在ThreadPoolExecutor中，它的全部代码如下：

```
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock();
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

task就是实际要执行的任务本体。有趣的是runWorker()方法会在task.run()开始前添加beforeExecute(wt, task)，而在task.run()执行完成后执行afterExecute(task, thrown)。这样线程池就可以在任务开始前及结束后插入一些套路化的东西。这两个方法均属于ThreadPoolExecutor，它们的代码如下：

```
protected void beforeExecute(Thread t, Runnable r) { }

protected void afterExecute(Runnable r, Throwable t) { }
```

这是两个权限为protected的空方法。很显然，Java API其实没什么套路，不过它为使用者留好了途径，使得他们可以通过继承ThreadPoolExecutor并重写这两个方法的方式实现自身的套路。

这实在是一个很有用的功能。我通常用它来更精细化的记录线程池及其中线程的状态变化，这对日志分析极为有益。

beforeExecute及afterExecute实际上是在每个任务开始前及结束后插入的AOP方法，通常它们会配合ThreadPoolExecutor中的另一个方法一起被重写：

```
protected void terminated() { }
```

terminated默认也是空方法，它会在线程池关闭时被触发。我们不妨以ThreadPoolExecutor中最常用的shutdown()方法为起点，来看一下调用terminated()的方法链。

首先shutdown()方法的代码为：

```
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(SHUTDOWN);
        interruptIdleWorkers();
        onShutdown();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
}
```

然后我们进入ThreadPoolExecutor.tryTerminate()：

```
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
        if (workerCountOf(c) != 0) {
            interruptIdleWorkers(ONLY_ONE);
            return;
        }

        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated();    // AOP方法
                } finally {
                    ctl.set(ctlOf(TERMINATED, 0));
                    termination.signalAll();
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
    }
}
```

在其内部可以找到terminated()。

下面给出一个应用beforeExecute，afterExecute，terminated的小例子：

```
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Test {

    static Long BEGIN = System.currentTimeMillis();

    public static void main(String[] args) throws InterruptedException {
        MyPool myPool = new MyPool(5, 5, 0L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());
        for (int i = 0; i < 5; i++) {
            myPool.execute(new MyTask("task" + i));
            Thread.sleep(10);
        }
        myPool.shutdown();
    }
}

class MyTask implements Runnable {

    String name;

    MyTask(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]{" + Thread.currentThread().getId() + "}" + this.name + " start...");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]{" + Thread.currentThread().getId() + "}" + this.name + " finish");
    }
}

class MyPool extends ThreadPoolExecutor {

    MyPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]=======AOP=======prepare work for " + ((MyTask)r).name);
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]=======AOP=======after work for " + ((MyTask)r).name);
    }

    @Override
    protected void terminated() {
        System.out.println("[" + (System.currentTimeMillis() - Test.BEGIN) + "]=======AOP=======after work for myPool shutdown");
    }
}
```

输出：

```
[4]=======AOP=======prepare work for task0
[4]{9}task0 start...
[14]=======AOP=======prepare work for task1
[14]{10}task1 start...
[24]=======AOP=======prepare work for task2
[24]{11}task2 start...
[34]=======AOP=======prepare work for task3
[34]{12}task3 start...
[44]=======AOP=======prepare work for task4
[44]{13}task4 start...
[1005]{9}task0 finish
[1005]=======AOP=======after work for task0
[1014]{10}task1 finish
[1014]=======AOP=======after work for task1
[1024]{11}task2 finish
[1024]=======AOP=======after work for task2
[1035]{12}task3 finish
[1036]=======AOP=======after work for task3
[1044]{13}task4 finish
[1045]=======AOP=======after work for task4
[1045]=======AOP=======after work for myPool shutdown
```

# 合理选择线程池中线程的数量

纯粹从性能的角度来看，线程数的极限值为可用CPU的数量。Java API提供了获取可用CPU的方法：

```
public class Test {

    public static void main(String[] args) {
        System.out.println(Runtime.getRuntime().availableProcessors());
    }
}
```

输出：

```
4
```

因为一个CPU同时只能运行一个线程，因此设定多于CPU数的线程通常都是伪并行，不仅无法得到性能的提升，反而还会因CPU在线程间的切换而导致性能的下降。

当然，这只是理论上的数值。设置线程数这种东西最重要的还是经验。通常有以下两种情况会导致我们设置多于上文极限值的线程数：

1. 从逻辑建模的角度而言，就需要设置这么多的线程。此时虽然会损失机器性能，但是却能更好的模拟现实环境，利于程序的开发及维护。

2. 生成的线程并非是一直占据CPU的，在它的生存周期中可能会让出CPU做一些无需CPU的操作(例如各种I/O中断)。此时适当的提高线程数可以提高系统性能。

# 查看线程池中线程的堆栈信息

我们先来看一个小例子：

```
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Test {

    public static void main(String[] args) {
        ThreadPoolExecutor te = new ThreadPoolExecutor(5, 5, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());
        for (int i = 0; i < 5; i++) te.submit(new MyTask("task" + i, 100, i));
        te.shutdown();
    }
}

class MyTask implements Runnable {

    String name;

    int d1;

    int d2;

    MyTask(String name, int d1, int d2) {
        this.name = name;
        this.d1 = d1;
        this.d2 = d2;
    }

    @Override
    public void run() {
        System.out.println("[" + Thread.currentThread().getId() + "]" + this.name + " result=" + (this.d1 / d2));
    }
}
```

输出：

```
[10]task1 result=100
[12]task3 result=33
[11]task2 result=50
[13]task4 result=25
```

这可真是一个扎心的结果：我们提交了5个任务，最后只有4组输出。很显然有一组因为除0异常导致线程崩了。之所以说这个结果扎心，并非因为抛出了异常，而在于系统对于异常无动于衷：单看输出天下太平，并没抛出任何异常。

这真的是最为糟心的情况了：程序出问题了，但是从日志来看到处都是正常的。一种最简单的，可以让我们获得部分异常堆栈的做法是用execute替换submit：

```
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Test {

    public static void main(String[] args) {
        ThreadPoolExecutor te = new ThreadPoolExecutor(5, 5, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());
        for (int i = 0; i < 5; i++) te.execute(new MyTask("task" + i, 100, i));
        te.shutdown();
    }
}

class MyTask implements Runnable {

    String name;

    int d1;

    int d2;

    MyTask(String name, int d1, int d2) {
        this.name = name;
        this.d1 = d1;
        this.d2 = d2;
    }

    @Override
    public void run() {
        System.out.println("[" + Thread.currentThread().getId() + "]" + this.name + " result=" + (this.d1 / d2));
    }
}
```

此时的输出为：

```
[11]task2 result=50
[12]task3 result=33
[10]task1 result=100
[13]task4 result=25
Exception in thread "pool-1-thread-1" java.lang.ArithmeticException: / by zero
	at com.test.MyTask.run(Test.java:32)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:745)
```

好多了，虽然无法精确定位，但是起码让我们知道线程池中有线程出异常了。

原因其实很好理解，submit与execute最大的不同之处就在于submit有返回值，而execute没有。因此execute必须要将线程的异常信息抛出，否则就是彻底没了。而submit的异常信息实际上是在它的结果中：

```
import java.util.concurrent.ExecutionException;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Test {

    public static void main(String[] args) {
        ThreadPoolExecutor te = new ThreadPoolExecutor(5, 5, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());
        for (int i = 0; i < 5; i++) {
            try {
                System.out.println(te.submit(new MyTask("task" + i, 100, i)).get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
            
        }
        te.shutdown();
    }
}

class MyTask implements Runnable {

    String name;

    int d1;

    int d2;

    MyTask(String name, int d1, int d2) {
        this.name = name;
        this.d1 = d1;
        this.d2 = d2;
    }

    @Override
    public void run() {
        System.out.println("[" + Thread.currentThread().getId() + "]" + this.name + " result=" + (this.d1 / d2));
    }
}
```

输出：

```
java.util.concurrent.ExecutionException: java.lang.ArithmeticException: / by zero
	at java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.util.concurrent.FutureTask.get(FutureTask.java:188)
	at com.test.Test.main(Test.java:14)
Caused by: java.lang.ArithmeticException: / by zero
	at com.test.MyTask.run(Test.java:40)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
	at java.util.concurrent.FutureTask.run(FutureTask.java:262)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:745)
[10]task1 result=100
null
[11]task2 result=50
null
[12]task3 result=33
null
[13]task4 result=25
null
```

此时除0异常在get()时被捕获，随后该方法向上层抛出ExecutionException，进而被我们的程序捕获。

虽然上述两种做法能获得部分异常堆栈，但这"部分信息"对于我们的异常分析基本是然并卵的。上述异常信息只能定位到核心Runnable的位置(MyTask.run)，但却定位不到是哪个线程池抛出的异常，无法提供线程池的详细信息。

对此，我们同样可以通过继承ThreadPoolExecutor获得更多的堆栈信息：

```
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.Future;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Test {

    public static void main(String[] args) {
        MyPool mp = new MyPool(5, 5, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());
        for (int i = 0; i < 5; i++) mp.execute(new MyTask("task" + i, 100, i));
        mp.shutdown();
    }
}

class MyPool extends ThreadPoolExecutor {

    MyPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    public String toString() {
        return "pool=" + this.getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

    public void execute(MyTask task) {
        super.execute(this.hull(task, this.baseTrace(task)));
    }

    public Future<?> submit(MyTask task) {
        return super.submit(this.hull(task, this.baseTrace(task)));
    }

    private Exception baseTrace(MyTask task) {
        return new Exception("[" + Thread.currentThread().getId() + "]" + this.toString() + "-------task=" + task.name);
    }

    private Runnable hull(final MyTask task, final Exception baseTrace) {
        return new Runnable() {
            @Override
            public void run() {
                try {
                    task.run();
                } catch (Exception e) {
                    baseTrace.printStackTrace();
                    throw e;
                }
            }
        };
    }
}

class MyTask implements Runnable {

    String name;

    int d1;

    int d2;

    MyTask(String name, int d1, int d2) {
        this.name = name;
        this.d1 = d1;
        this.d2 = d2;
    }

    @Override
    public void run() {
        System.out.println("[" + Thread.currentThread().getId() + "]" + this.name + " result=" + (this.d1 / d2));
    }
}
```

输出：

```
[10]task1 result=100
[12]task3 result=33
[11]task2 result=50
[13]task4 result=25
java.lang.Exception: [1]pool=com.test.MyPool@470ae2bf-------task=task0
	at com.test.MyPool.baseTrace(Test.java:38)
	at com.test.MyPool.execute(Test.java:30)
	at com.test.Test.main(Test.java:13)
Exception in thread "pool-1-thread-1" java.lang.ArithmeticException: / by zero
	at com.test.MyTask.run(Test.java:72)
	at com.test.MyPool$1.run(Test.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:745)
```

该输出中同时打出了出错的线程池及任务。一般是不需要打出任务的：因为通常我们只会向线程池提交一种任务。

事实上，这并不能被称之为一个功能，这只是一个小技巧而已：我们仅仅只是在默认的ThreadPoolExecutor之外又包了一层外壳，外壳中记录下当前线程池的信息(如上文代码所示，如果有必要也可以特化外壳类中execute，submit等接受的Runnable的类型，这样就可以打印出更多的任务相关的信息)，当发生异常时，将该线程池的堆栈信息打印出来。

同理，使用submit时的代码为：

```
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class Test {

    public static void main(String[] args) {
        MyPool mp = new MyPool(5, 5, 0, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());
        for (int i = 0; i < 5; i++) {
            try {
                System.out.println(mp.submit(new MyTask("task" + i, 100, i)).get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
            
        }
        mp.shutdown();
    }
}

class MyPool extends ThreadPoolExecutor {

    MyPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    public String toString() {
        return "pool=" + this.getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

    public void execute(MyTask task) {
        super.execute(this.hull(task, this.baseTrace(task)));
    }

    public Future<?> submit(MyTask task) {
        return super.submit(this.hull(task, this.baseTrace(task)));
    }

    private Exception baseTrace(MyTask task) {
        return new Exception("[" + Thread.currentThread().getId() + "]" + this.toString() + "-------task=" + task.name);
    }

    private Runnable hull(final MyTask task, final Exception baseTrace) {
        return new Runnable() {
            @Override
            public void run() {
                try {
                    task.run();
                } catch (Exception e) {
                    baseTrace.printStackTrace();
                    throw e;
                }
            }
        };
    }
}

class MyTask implements Runnable {

    String name;

    int d1;

    int d2;

    MyTask(String name, int d1, int d2) {
        this.name = name;
        this.d1 = d1;
        this.d2 = d2;
    }

    @Override
    public void run() {
        System.out.println("[" + Thread.currentThread().getId() + "]" + this.name + " result=" + (this.d1 / d2));
    }
}
```

输出为：

```
java.lang.Exception: [1]pool=com.test.MyPool@620a9239-------task=task0
	at com.test.MyPool.baseTrace(Test.java:46)
	at com.test.MyPool.submit(Test.java:42)
	at com.test.Test.main(Test.java:16)
java.util.concurrent.ExecutionException: java.lang.ArithmeticException: / by zero
	at java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.util.concurrent.FutureTask.get(FutureTask.java:188)
	at com.test.Test.main(Test.java:16)
Caused by: java.lang.ArithmeticException: / by zero
	at com.test.MyTask.run(Test.java:80)
	at com.test.MyPool$1.run(Test.java:54)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
	at java.util.concurrent.FutureTask.run(FutureTask.java:262)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:745)
[10]task1 result=100
null
[11]task2 result=50
null
[12]task3 result=33
null
[13]task4 result=25
null
```

# 同一个线程池execute()的Runnable可以不同

通常情况下，我们只会向一个线程池提供一种Runnable，即该线程池产生的线程的能力都相同。这是Java推荐的使用方式，一个线程池只产生单一功能的线程有利于程序逻辑的划分。但是这仅仅是推荐，而非必须。事实上，我们可以向线程池提交任意的，不同的Runnable：

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(1);
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("r1");
            }
        };
        Runnable r2 = new Runnable() {
            @Override
            public void run() {
                System.out.println("r2");
            }
        };
        for (int i = 0; i < 4; i++) {
            if (i % 2 == 0) es.execute(r1);
            else es.execute(r2);
        }
        es.shutdown();
    }
}
```

输出：

```
r2
r1
r2
r1
```

可以这样理解：线程池管理的只是线程的壳子，而Thread与Runnable是解耦的。我们可以将Thread看作人的肉体，Runnable看作人的灵魂。线程池管理的仅仅是人的肉体，在需要使用时赋予不同的灵魂(上例中的线程池只管理了一个线程)。