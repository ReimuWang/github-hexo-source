---
title: Java 并发-Fork Join框架
date: 2018-01-23 16:27:36
tags: [Java,并发]
categories: Java 并发
---

"分治法"是一种非常有效的解决问题的思路，大名鼎鼎的MapReduce就是它的实际应用。

我们可以以一个小例子来简单介绍下分治法的基本思想：假如我们需要处理1000个数据，而这1000个数据的处理是彼此独立的，不需要建立特定的处理顺序。比较极端的方式有两种：其一是同时处理这1000个数据，这样做效率最高，但对系统的负担也最大。其二是每次只处理1个，处理1000次，这样做对系统的负担最小，但效率最低。

<!-- more -->

其实这两种处理方式可以看作同一个模式下的两个极端情况：每次处理x个，处理y轮，保证x*y为数据总数即可。其实这已经运用了分治法的思想，将一个大的问题，分解成了y轮。

因此分治法的本质就是分解：将无法直接解决的大的问题分解为一个个小的，可以通过现有资源解决的问题。在解决这些小的问题后再进行汇总，最终间接解决大的问题。其实不仅仅是编程，为人处事也是这个道理。

具体到Java API中，分治法的应用便是Fork Join框架。Fork的原义是叉子，引申义就是问题的分解：像叉子的前端一样被分解为多个更小的问题。事实上，Java的这种起名方式源自Linux。在Linux中，fork()函数用来创建子进程。

而Join则与Java API中的join方法相同，代表等待。

因此Fork Join确实是很形象的描述了分治法的精髓：当问题需要分解时，它便像叉子的前端一样被分解为多个分叉，而先完成的分叉会等待尚未完成的分叉，待全部分叉均完成后再汇聚起来，向下进行。

依然以开篇的那个小例子为例，fork出1000个叉确实是太多了，而只fork出一个叉相当于就没分治。因此合乎逻辑的做法应当是根据系统的承载能力及对性能的要求fork出一个(1,1000)之间的整数。很显然，这里需要用到线程池。

这个线程池名为java.util.concurrent.ForkJoinPool，它的类定义如下：

```
public class ForkJoinPool extends AbstractExecutorService
```

果不其然，它继承了AbstractExecutorService，承接自线程池一脉。在[Java 并发-线程池](/2017/10/03/Java 并发-线程池/)中，我们曾给出过线程池核心类/接口的层次关系图：

![0.jpg](/images/blog_pic/Java 并发/Fork Join框架/0.jpg)

现在，我们可以进一步丰富这张图：

![1.jpg](/images/blog_pic/Java 并发/Fork Join框架/1.jpg)

它的常用构造函数有两个：

```
public ForkJoinPool(int parallelism) {
    this(parallelism, defaultForkJoinWorkerThreadFactory, null, false);
}

public ForkJoinPool() {
    this(Runtime.getRuntime().availableProcessors(),
         defaultForkJoinWorkerThreadFactory, null, false);
}
```

前者会按照使用者的要求创建一个大小为parallelism的线程池，而后者则会直接取可用CPU数作为线程池的大小。二者内部实际上调用的是同一个方法：

```
public ForkJoinPool(int parallelism,
                    ForkJoinWorkerThreadFactory factory,
                    Thread.UncaughtExceptionHandler handler,
                    boolean asyncMode) {
    checkPermission();
    if (factory == null)
        throw new NullPointerException();
    if (parallelism <= 0 || parallelism > MAX_ID)
        throw new IllegalArgumentException();
    this.parallelism = parallelism;
    this.factory = factory;
    this.ueh = handler;
    this.locallyFifo = asyncMode;
    long np = (long)(-parallelism);
    this.ctl = ((np << AC_SHIFT) & AC_MASK) | ((np << TC_SHIFT) & TC_MASK);
    this.submissionQueue = new ForkJoinTask<?>[INITIAL_QUEUE_CAPACITY];
    int n = parallelism << 1;
    if (n >= MAX_ID)
        n = MAX_ID;
    else {
        n |= n >>> 1; n |= n >>> 2; n |= n >>> 4; n |= n >>> 8;
    }
    workers = new ForkJoinWorkerThread[n + 1];
    this.submissionLock = new ReentrantLock();
    this.termination = submissionLock.newCondition();
    StringBuilder sb = new StringBuilder("ForkJoinPool-");
    sb.append(poolNumberGenerator.incrementAndGet());
    sb.append("-worker-");
    this.workerNamePrefix = sb.toString();
}
```

很长，而且看起来做了很多事，挺复杂的样子。不过我们很简单就可以确定的是，这确实是在构建一个和我们最常用的ThreadPoolExecutor完全不同的线程池，并且这个线程池应该是比ThreadPoolExecutor要复杂得多。

事实上也的确如此，不过这种复杂对于使用者基本是透明的，因为它们绝大多数都被用以实现ForkJoinPool更复杂的逻辑需求以及优化它的性能。例如，对于普通的线程池，也就是ThreadPoolExecutor而言，提交给它的任务可以是不同的，因此两个线程之间是无法互相帮助的。而ForkJoinPool中的任务都是相同的(叉子的每个尖端当然都是相同的)，因此两个线程间可以互相帮助：例如线程1已将线程池分配给它的任务全部执行完，而线程2的阻塞队列中尚有任务积压，那么线程1就可以帮着线程2完成一部分积压的任务。当然，这里只是简单说下思路，实际实现起来还是比较麻烦的。例如，为了避免在帮助时发生冲突，从自身队列中取数据时应取队首的，而帮助他人时则从队尾开始拿数据。

再比如，ForkJoinPool会使用一个无锁的栈来管理空闲线程。如果一个工作线程没有被分配任务，那么它除了帮助他人外，还有可能(注意仅仅只是可能)被挂起，被挂起的线程将会被压入ForkJoinPool所管理的那个栈中，待需要时再唤醒线程。

在任务的提交上，ForkJoinPool最常用的提交方法为：

```
public <T> ForkJoinTask<T> submit(ForkJoinTask<T> task) {
    if (task == null)
        throw new NullPointerException();
    forkOrSubmit(task);
    return task;
}
```

从入参所属类的名字上我们便可看出，这个方法是为ForkJoinPool量身打造的，而java.util.concurrent.ForkJoinTask的类定义为：

```
public abstract class ForkJoinTask<V> implements Future<V>, Serializable
```

它是ForkJoinPool专用的任务，其设计思路与我们最熟悉的构成Thread的核心Runnable完全不同，不过二者的性质是相同的，都代表要线程池执行的任务本身，其内部封装着具体的任务逻辑。

我们在使用时通常会使用ForkJoinTask的如下两个方法：

```
public final ForkJoinTask<V> fork()

public final V join()
```

fork及join不必多说，它们所实现的就是Fork Join的核心功能，join返回任务的执行结果，其类型就是ForkJoinTask类定义中的V。

ForkJoinTask是一个抽象类，我们常用的它的子类有两个，它们的类定义分别为：

```
public abstract class RecursiveTask<V> extends ForkJoinTask<V>
```

```
public abstract class RecursiveAction extends ForkJoinTask<Void>
```

这又是两个抽象类，也就是说摆明了是要我们继承了才能用。

理所当然的，它们都位于java.util.concurrent包中。需要注意的是，单看ForkJoinTask，仿佛它的子类都应有返回值，但事实上，Java API不仅提供了有返回值的子类(RecursiveTask，类比于Callable)，同时也提供了无返回值的子类(RecursiveAction，类比于Runnable)。

无论是RecursiveTask，亦或是RecursiveAction，执行任务的逻辑都被封装在了方法compute()中：

有意思的是，这个方法并未出现在ForkJoinTask中，也就是RecursiveTask及RecursiveAction特有的执行方法，相当于Runnable的run()，或是Callable的call()。事实上，ForkJoinTask确实不仅仅这两个子类，我并未关注其他子类是怎么实现的，不过估计应该是有子类不依靠compute()来执行吧。

这个compute()在RecursiveTask的代码为：

```
protected abstract V compute();
```

在RecursiveAction中的代码为：

```
protected abstract void compute();
```

自然，这便是需要我们自行实现，封装任务逻辑的关键代码(也是RecursiveTask及RecursiveAction中唯一的抽象方法)。

无论分治法在内部将问题分解为了多少个小问题，从外部调用者来看，只需提交一个任务，然后等待ForkJoinPool得到该问题的解，这个解包含在submit的返回值ForkJoinTask中，它对应未经分解的那个大问题，其值可以通过它的get方法得到：

```
public final V get() throws InterruptedException, ExecutionException
```

最后再提一点，既然ForkJoinTask从概念上对应于Runnable或Callable，那么它也该有个类似于Thread一样的容器来封装才对。没错，这个容器名为java.util.concurrent.ForkJoinWorkerThread，它的类定义为：

```
public class ForkJoinWorkerThread extends Thread
```

看来较之于任务，容器倒是没那么另类，是直接继承了最常见的Thread。

下面我们来看一个小例子：要求做1-200000的数列求和，假设当前系统一个线程单次最多只能计算10000个数：

```
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;

public class Test {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        MyTask myTask = new MyTask(1L, 200000L);
        ForkJoinTask<Long> forkJoinTask = forkJoinPool.submit(myTask);
        System.out.println(forkJoinTask.get());
    }
}

class MyTask extends RecursiveTask<Long> {

    private static final long serialVersionUID = 1L;

    private static final int LIMIT = 10000;

    private long begin;

    private long end;

    MyTask(long begin, long end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Long compute() {
        long sum = 0L;
        long count = this.end - this.begin + 1;
        if (count <= MyTask.LIMIT) {
            for (long i = this.begin; i <= this.end; i++) sum += i;
            return sum;
        }
        long mid = this.begin + (this.end - this.begin) / 2;    // 将规模缩减至一半

        MyTask myTask1 = new MyTask(this.begin, mid);
        myTask1.fork();
        sum += myTask1.join();

        MyTask myTask2 = new MyTask(mid + 1, this.end);
        myTask2.fork();
        sum += myTask2.join();
        
        return sum;
    }
}
```

输出：

```
20000100000
```

看完这个小例子，想必很自然的就会联想到递归：因为这与递归实在是太像了。没错，事实上，大家不妨认真思考下，递归也是分治思想的体现啊！

不过，递归默认是不支持并发的，如果想将原生的递归改造为并发也是非常麻烦的。所以我们其实也可以这样想：Fork Join框架可以看作是并发环境下的递归。

既然Fork Join框架本质上和递归类似，那么如果调用的层次过深，它也同样可能出现栈溢出。同时，由于Fork Join底层使用的是线程池，那么它也可能出现普通线程池容易引发的错误：例如程序占用的线程数量过多，导致系统性能下降甚至崩溃。这都是使用Fork Join框架时需要注意的地方。