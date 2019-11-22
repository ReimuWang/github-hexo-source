---
title: Java 并发-线程组
date: 2017-10-06 11:10:36
tags: [Java,并发,线程组]
categories: Java 并发
---

# 线程组与线程池的区别

<!-- more -->

线程组关注的是线程在逻辑上的从属，使用线程组可让程序的逻辑结构更清晰，更利于模拟复杂的现实环境；线程池关注的是线程的管理，使用线程池可以避免线程频繁的创建与关闭，提高系统性能。

美好的猜想：从线程池处获得线程，而后交由线程组进行业务上的划分管理。

实际上：向线程池提交的是Runnable，具体Thread是由线程池内部创建的。而关联Thread与ThreadGroup需要在新建Thread时显示声明。因此当前尚未找到能让二者协调工作的方法。

或者说，这个猜想本质上是错误的：线程池管理的并非一个个已经成型的线程。即用以构建线程池中的线程所需提供的素材并非线程本身，而是生成线程的核心部件Runnable。也就是说线程池并不希望我们确定下来线程再交给它，这样它也就没法管理了。使用线程池时我们只管从池子里取线程即可，该线程是基于我们传入的Runnable生成的，因此可以满足我们的业务需求，至于这个线程是何时生成的，怎么生成的我们一概不管。而线程组是在明确的给线程打上编号：即某某线程属于某某组。一定要先有一个明确的线程才行。因此二者是无法协同工作的。

线程池适用的场景：大量的量产型工人迅速的出生死亡做着流程化的工作，我们不关心干活的工人是谁，甚至于我们认为新出生的工人其实不是新出生的，而是一个我们以为已经死了的工人其实没死复用的，这都无所谓，只要核心部件Runnable相同，线程池创造出的工人能力就是相同的，他们都可以毫不违和的做其他工人的工作，他们没有"自我"，没有某个工作只有这个线程才能做的情况，他们之间可以互相替代。

线程组适用的场景：固定的几个精英人士长时间生存，他们的工作是独一无二的，只有他们才能做，因此比起他们做的事，对他们本身的管理更为重要，因为他们的人和他们做的事是一一对应的，管理了他们的人，自然相当于管理了他们做的事。

下面的示例说明了这个问题。

# 线程组示例

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " start...");
        synchronized(Test.class) {
            try {
                Test.class.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private static void showThreadGroup(ThreadGroup threadGroup) {
        System.out.println("=========== " + threadGroup.getName() + " ===========");
        // 已启动而未终止的线程数，阻塞状态或等待状态均算。
        // 由于线程是动态的，因此这个值只是估计值，无法保证精确。
        System.out.println("当前活跃线程数" + threadGroup.activeCount());
        System.out.println("包含线程信息:");
        threadGroup.list();
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadGroup hongMoGuan = new ThreadGroup("红魔馆");
        ThreadGroup yongYuanTing = new ThreadGroup("永远亭");
        for (String name : new String[]{"蕾米莉亚", "芙兰朵露", "帕秋莉"}) new Thread(hongMoGuan, new Test(), name).start();
        for (String name : new String[]{"蓬莱山辉夜", "八意永琳", "铃仙"}) new Thread(yongYuanTing, new Test(), name).start();
        Thread.sleep(10);
        Test.showThreadGroup(hongMoGuan);
        Test.showThreadGroup(yongYuanTing);
        Thread.sleep(10);
        System.out.println("=========== 罪袋在工作 ===========");
        ExecutorService zuidaiEs = Executors.newFixedThreadPool(3);
        Runnable zuidai = new Runnable() {
            @Override
            public void run() {
                System.out.println("罪袋 " + Thread.currentThread().getId() + " 完成工作，被回收...");
            }
        };
        // 需要干出10个罪袋的工作量的活。
        // 我们不关心到底有几个罪袋真正工作了：也许有10个罪袋各干了一人份的活；也许只有两个罪袋每人干了5人份。
        // 只要结果上干完了就行。
        for (int i = 0; i < 10; i++) zuidaiEs.submit(zuidai);
    }
}
```

输出结果：

```
芙兰朵露 start...
蕾米莉亚 start...
蓬莱山辉夜 start...
八意永琳 start...
铃仙 start...
帕秋莉 start...
=========== 红魔馆 ===========
当前活跃线程数3
包含线程信息:
java.lang.ThreadGroup[name=红魔馆,maxpri=10]
    Thread[蕾米莉亚,5,红魔馆]
    Thread[芙兰朵露,5,红魔馆]
    Thread[帕秋莉,5,红魔馆]
=========== 永远亭 ===========
当前活跃线程数3
包含线程信息:
java.lang.ThreadGroup[name=永远亭,maxpri=10]
    Thread[蓬莱山辉夜,5,永远亭]
    Thread[八意永琳,5,永远亭]
    Thread[铃仙,5,永远亭]
=========== 罪袋在工作 ===========
罪袋 15 完成工作，被回收...
罪袋 17 完成工作，被回收...
罪袋 17 完成工作，被回收...
罪袋 16 完成工作，被回收...
罪袋 16 完成工作，被回收...
罪袋 17 完成工作，被回收...
罪袋 17 完成工作，被回收...
罪袋 15 完成工作，被回收...
罪袋 17 完成工作，被回收...
罪袋 16 完成工作，被回收...
```

# 终止线程组中的线程(stop)

ThreadGroup类提供如下方法：

```
public final void stop()
```

类似于Thread类提供的stop()，ThreadGroup类提供的stop()会强制终止线程组中的所有线程。理所当然的该方法也会有Thread类中stop()类似的问题。因此和Thread类中的stop()一样，该方法也是废弃方法。