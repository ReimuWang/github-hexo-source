---
title: Java 并发-信号量Semaphore
date: 2018-01-08 14:55:36
tags: [Java,并发,Semaphore,信号量]
categories: Java 并发
---

无论是synchronized抑或是Lock，同一时刻只允许有1个线程访问临界区，如果我们要实现操作系统中的多信号量的功能，即同一时刻允许复数个线程访问临界区，则需要使用Java API提供的java.util.concurrent.Semaphore类。该类常用的构造函数有以下两个：

<!-- more -->

```
/**
 * permits, int 信号量数
 */
public Semaphore(int permits)

/**
 * permits, int 信号量数
 * fair, boolean true--公平 false--不公平
 */
public Semaphore(int permits, boolean fair)
```

通常情况下，每个线程只会申请一个信号量，此时信号量数就相当于同时能访问临界区的线程数。Semaphore类的常用方法如下：

```
/**
 * 申请获得一个信号量
 */
public void acquire() throws InterruptedException

/**
 * 申请获得一个信号量，但不响应中断
 */
public void acquireUninterruptibly()

/**
 * 申请获得一个信号量。成功返回true，失败不会等待，直接返回false
 */
public boolean tryAcquire()

/**
 * 申请获得一个信号量。
 * 成功返回true
 * 失败后会等待一段时间(timeout)，若在此期间获得信号量则返回true，反之返回false
 */
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException

/**
 * 释放已获得的信号量
 */
public void release()
```

显然这与Lock中的加锁解锁极为类似。下面我们来看一个小例子：

```
import java.util.Calendar;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class Test {

    private static Semaphore SEMP = new Semaphore(5);

    public static void main(String[] args) {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                try {
                    Test.SEMP.acquire();
                    Thread.sleep(1000);
                    System.out.println(Calendar.getInstance().get(Calendar.SECOND) + "-----" + Thread.currentThread().getId() + " done.");
                    Test.SEMP.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        int threadCount = 20;
        ExecutorService es = Executors.newFixedThreadPool(threadCount);
        for (int i = 0; i < threadCount; i++) es.submit(r);
        es.shutdown();
    }
}
```

输出：

```
50-----12 done.
50-----10 done.
50-----9 done.
50-----13 done.
50-----11 done.
51-----14 done.
51-----17 done.
51-----21 done.
51-----18 done.
51-----16 done.
52-----19 done.
52-----20 done.
52-----15 done.
52-----22 done.
52-----23 done.
53-----25 done.
53-----26 done.
53-----24 done.
53-----27 done.
53-----28 done.
```

分析输出可得：每秒输出1组，每组5个。与预期相符。