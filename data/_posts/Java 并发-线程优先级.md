---
title: Java 并发-线程优先级
date: 2017-09-29 17:45:36
tags: [Java,并发,线程,优先级]
categories: Java 并发
---

用户可指定不同线程间的优先级，高优先级的线程在竞争资源时更有利。注意这仅仅是"有利"，即高优先级线程较之低优先级线程有更高的概率获得资源，而非在资源争夺中一定会获胜。Java将线程的优先级分为1-10的十个级别，并且提供了3个内置在Thread类中的静态标量：

```
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
```

默认情况下，线程的优先级为NORM_PRIORITY = 5。

<!-- more -->

Java中的线程优先级是一个相对不靠谱的东西：

- 但凡是牵扯到概率，程序就无法精确控制。我们并不能确定高优先级的线程就一定会先于低优先级的线程获得资源。

- 本质上，Java线程的优先级必须要映射到操作系统底层的线程优先级才能发挥作用。但是很多操作系统的线程优先级的分级根本就没有10个那么多。这就导致复数个Java层面的线程优先级在操作系统看来实际是一回事。即程序员将两个线程的优先级分别设置为5,6并期望二者能在优先级上有所差异，但其实底层操作系统在实现时也许二者的优先级就完全一样了。这也是Java提供3个静态标量的原因：无论操作系统多么糟心，3个档次的优先级一般还是有的。因此推荐用静态标量的方式设置优先级，而不要自己指定数字。

所以，在要求严格的场合，还是要程序员自己在应用层面通过代码逻辑来控制线程的优先级。

小例子：

```
public class Test extends Thread {

    public Test(String name) {
        super(name);
    }

    @Override
    public void run() {
        int count = 0;
        while (true) {
            synchronized (Test.class) {
                count++;
                if (count > 10000000) {
                    System.out.println(Thread.currentThread().getName() + " finish.");
                    break;
                }
            }
        }
    }

    public static void main(String[] args) {
        Thread low = new Test("low");
        low.setPriority(Thread.MIN_PRIORITY);
        Thread high = new Test("high");
        high.setPriority(Thread.MAX_PRIORITY);
        low.start();
        high.start();
    }
}
```

绝大多数情况，输出如下：

```
high finish.
low finish.
```