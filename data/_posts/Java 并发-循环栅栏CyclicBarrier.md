---
title: Java 并发-循环栅栏CyclicBarrier
date: 2018-01-08 17:29:36
tags: [Java,并发,CyclicBarrier]
categories: Java 并发
---

CyclicBarrier可以看作是[Java 并发-倒计数器CountDownLatch](/2018/01/08/Java 并发-倒计数器CountDownLatch/)的功能增强版。其中Barrier(栅栏)是Latch(门栓)的同义词，本质上都表示一种障碍。CyclicBarrier可理解为"循环的倒计数门栓"。即CyclicBarrier在CountDownLatch的基础上加入了循环的功能：每当一次计数结束，系统都会执行一个固定的程序员设定好的操作。随后将计数值归为初始值，再开始下一次循环。

<!-- more -->

java.util.concurrent.CyclicBarrier的类定义为：

```
public class CyclicBarrier
```

其常用构造函数为：

```
public CyclicBarrier(int parties, Runnable barrierAction)
```

其中parties是倒计数值，barrierAction是每次计数结束后要进行的操作。

我们不妨模拟这样1个场景：有一个化学实验室，同时可供3名学生作为一个小组进行实验。要求必须凑够3人才能开始实验。上一组实验完成后下一组才可进入。组队的学生没有特殊的要求，只要凑齐了3人，不管是谁都可以组队进行实验。现有9名学生会逐渐去实验室做实验。

该场景的代码实现为：

```
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class Test {

    static int TEAM_SIZE = 3;

    private static long BASE_BEGIN = System.currentTimeMillis();

    private static CyclicBarrier CB;

    public static void main(String[] args) {
        Runnable barrierAction = new Runnable() {
            @Override
            public void run() {
                System.out.println("=====一组学生在实验室中完成实验=====");
            }
        };
        Test.CB = new CyclicBarrier(Test.TEAM_SIZE, barrierAction);
        Runnable student = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(new Random().nextInt(10000));
                    System.out.println(Thread.currentThread().getName() + "于第" + (System.currentTimeMillis() - Test.BASE_BEGIN) + "毫秒到达实验室");
                    Test.CB.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        };
        for (int i = 0; i < 9; i++) new Thread(student, "学生" + i).start();
    }
}
```

输出：

```
学生5于第1640毫秒到达实验室
学生7于第1819毫秒到达实验室
学生4于第6542毫秒到达实验室
=====一组学生在实验室中完成实验=====
学生0于第8260毫秒到达实验室
学生8于第8365毫秒到达实验室
学生3于第8816毫秒到达实验室
=====一组学生在实验室中完成实验=====
学生1于第9146毫秒到达实验室
学生2于第9709毫秒到达实验室
学生6于第9963毫秒到达实验室
=====一组学生在实验室中完成实验=====
```

如果我们将学生个数改为10：

```
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class Test {

    static int TEAM_SIZE = 3;

    private static long BASE_BEGIN = System.currentTimeMillis();

    private static CyclicBarrier CB;

    public static void main(String[] args) {
        Runnable barrierAction = new Runnable() {
            @Override
            public void run() {
                System.out.println("=====一组学生在实验室中完成实验=====");
            }
        };
        Test.CB = new CyclicBarrier(Test.TEAM_SIZE, barrierAction);
        Runnable student = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(new Random().nextInt(10000));
                    System.out.println(Thread.currentThread().getName() + "于第" + (System.currentTimeMillis() - Test.BASE_BEGIN) + "毫秒到达实验室");
                    Test.CB.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        };
        for (int i = 0; i < 10; i++) new Thread(student, "学生" + i).start();
    }
}
```

则输出：

```
学生0于第834毫秒到达实验室
学生4于第1119毫秒到达实验室
学生7于第2412毫秒到达实验室
=====一组学生在实验室中完成实验=====
学生3于第2671毫秒到达实验室
学生8于第6435毫秒到达实验室
学生6于第6929毫秒到达实验室
=====一组学生在实验室中完成实验=====
学生5于第7242毫秒到达实验室
学生9于第8087毫秒到达实验室
学生2于第8644毫秒到达实验室
=====一组学生在实验室中完成实验=====
学生1于第9065毫秒到达实验室
```

因未达到组队条件学生1将无法进行实验，且程序也无法终止。

通过上面的小例子，我们还可以注意到CyclicBarrier需检查两个异常：InterruptedException，BrokenBarrierException。其中InterruptedException是所有等待操作基本都会检查的，而BrokenBarrierException是CyclicBarrier特有的。因为在同一个计数周期内的线程之间是有连带关系的，若其中一个被中断了，剩下的即便没有被中断本次倒数也无效了，不能无限的等待下去，此时这些线程就会抛出BrokenBarrierException。

让我们来简化上面的小例子：

```
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class Test {

    static int TEAM_SIZE = 3;

    private static CyclicBarrier CB;

    public static void main(String[] args) {
        Runnable barrierAction = new Runnable() {
            @Override
            public void run() {
                System.out.println("=====一组学生在实验室中完成实验=====");
            }
        };
        Test.CB = new CyclicBarrier(Test.TEAM_SIZE, barrierAction);
        Runnable student = new Runnable() {
            @Override
            public void run() {
                try {
                    Test.CB.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        };
        for (int i = 0; i < 3; i++) {
            Thread t = new Thread(student, "学生" + i);
            t.start();
            if (i == 1) t.interrupt();
        }
    }
}
```

我们将线程总数控制为3，并中断线程1，则输出：

```
java.lang.InterruptedException
	at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:204)
	at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:355)
	at com.test.Test$2.run(Test.java:27)
	at java.lang.Thread.run(Thread.java:745)
java.util.concurrent.BrokenBarrierException
	at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:243)
	at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:355)
	at com.test.Test$2.run(Test.java:27)
	at java.lang.Thread.run(Thread.java:745)
java.util.concurrent.BrokenBarrierException
	at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:243)
	at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:355)
	at com.test.Test$2.run(Test.java:27)
	at java.lang.Thread.run(Thread.java:745)
```

和预期的一样，我们得到了一个InterruptedException及两个BrokenBarrierException。