---
title: Java 并发-哲学家就餐问题
date: 2018-05-23 17:18:36
tags: [Java,并发]
categories: Java 并发
---

在[Java 并发-并发级别](/2017/10/04/Java 并发-并发级别/)一文中，我们讨论过多线程的活跃性问题。其中最严重的一种情况就是死锁。关于死锁，在那篇文章中我们做过简要的描述，现摘录如下：

**死锁**

简单的小例子：线程A持有资源x而欲请求资源y。线程B持有资源y而欲请求资源x。不巧的是x，y均为临界区。如果现状得不到改善(AB均不愿意放弃已到手的资源)，那么死锁将一直进行下去，永远看不到解决的希望。因此死锁是活跃性问题中最糟糕的情况：它会直接导致被锁住的线程卡死。

<!-- more -->

哲学家就餐问题就是一个用于描述死锁的经典案例：

假设有5位哲学家围坐在1张圆桌旁，他们从不沟通，且只会做以下两件事之一：吃饭或者思考。并且吃饭的时候不思考，思考的时候不吃饭。餐桌正中有1大盘意大利面，每个哲学家左右手边各有一把叉子，因为1把餐叉很难吃到面，因此设定哲学家必须同时取到左右手的餐叉才能吃到面，并且我们还设定哲学家只会使用他所临近的那两把叉子。

5位哲学家，保险起见应该有10把叉子才行。但是现在只有5把叉子。这就存在着死锁的可能性了，例如每个哲学家都拿起来他们左手边的叉子，又都不愿意放下已获得的叉子，这样大家都吃不到饭，导致全部饿死。

我们也可以简化一下这个问题，即将哲学家减少至两人，相对的叉子也就只有两把了。问题的本质还是一样的，依然会有死锁的可能。下面我们就以两位哲学家的情况为例编码验证一下，假设双方均拿起了左手边的叉子，触发死锁：

```
package com.test;

public class Test extends Thread {

    private static final Object FORK1 = new Object();

    private static final Object FORK2 = new Object();

    private Object tool;

    private Test(Object tool) {
        this.tool = tool;
        if (this.tool == Test.FORK1) this.setName("哲学家1");
        if (this.tool == Test.FORK2) this.setName("哲学家2");
    }

    @Override
    public void run() {
        if ("哲学家1".equals(this.getName())) {
            synchronized (Test.FORK1) {
                try {
                    Thread.sleep(500);
                    synchronized (Test.FORK2) {
                        System.out.println(this.getName() + "获得所需的两把叉子，开始吃饭");
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        if ("哲学家2".equals(this.getName())) {
            synchronized (Test.FORK2) {
                try {
                    Thread.sleep(500);
                    synchronized (Test.FORK1) {
                        System.out.println(this.getName() + "获得所需的两把叉子，开始吃饭");
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        Test t1 = new Test(Test.FORK1);
        Test t2 = new Test(Test.FORK2);
        t1.start();
        t2.start();
    }
}
```

运行后，程序无法结束，陷入死锁。

在实际的生产工作中，如果某程序的相关进程不再工作，且CPU占用率为0或大幅下降(陷入死锁的线程是不会占用CPU的)，就要考虑是否有陷入死锁的可能性了。具体来说，我们可以使用JDK提供的一些小工具来检测死锁。

在[JVM-JDK自带JVM工具](/2017/11/03/JVM-JDK自带JVM工具/)一文中，我们曾经详细介绍过这些小工具，现在以上文哲学家死锁的代码为例，再次温习下它们的使用方法吧。

首先，我们使用jps获得进程的核心数据(主要是LVMID):

```
wangyikai1@5CD6227DBW MINGW64 /d/work/java/jdk/bin
$ ./jps -l
7540 sun.tools.jps.Jps
1312 D:\work\eclipse\\plugins/org.eclipse.equinox.launcher_1.3.0.v20140415-2008.jar
2244 com.test.Test
```

显然，其中的

```
2244 com.test.Test
```

就是我们要检测的进程啦！而2244是它的LVMID。

然后使用jstack来获得该进程的线程信息：

```
wangyikai1@5CD6227DBW MINGW64 /d/work/java/jdk/bin
$ ./jstack -l 2244 > /e/1.txt
```

此时线程信息被我重定向到了/e/1.txt中，赶紧来看下它里面的内容吧：

```
2018-05-24 10:38:44
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.80-b11 mixed mode):

"DestroyJavaVM" prio=6 tid=0x000000000253e800 nid=0x179c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"哲学家2" prio=6 tid=0x000000000c2a1800 nid=0x1a20 waiting for monitor entry [0x000000000cf3f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.test.Test.run(Test.java:36)
	- waiting to lock <0x00000007d5d9b028> (a java.lang.Object)
	- locked <0x00000007d5d9b038> (a java.lang.Object)

   Locked ownable synchronizers:
	- None

"哲学家1" prio=6 tid=0x000000000c29f000 nid=0x14ac waiting for monitor entry [0x000000000cd5f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.test.Test.run(Test.java:24)
	- waiting to lock <0x00000007d5d9b038> (a java.lang.Object)
	- locked <0x00000007d5d9b028> (a java.lang.Object)

   Locked ownable synchronizers:
	- None

"Service Thread" daemon prio=6 tid=0x000000000c284800 nid=0x870 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread1" daemon prio=10 tid=0x000000000c282800 nid=0x1b54 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread0" daemon prio=10 tid=0x000000000aea9000 nid=0x4c4 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Attach Listener" daemon prio=10 tid=0x000000000aecf000 nid=0xe10 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Signal Dispatcher" daemon prio=10 tid=0x000000000aeca000 nid=0x11f0 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Finalizer" daemon prio=8 tid=0x000000000ae52800 nid=0xcdc in Object.wait() [0x000000000c13f000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d5d04858> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	- locked <0x00000007d5d04858> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

   Locked ownable synchronizers:
	- None

"Reference Handler" daemon prio=10 tid=0x000000000ae51000 nid=0x18e8 in Object.wait() [0x000000000bf3f000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d5d04470> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:503)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	- locked <0x00000007d5d04470> (a java.lang.ref.Reference$Lock)

   Locked ownable synchronizers:
	- None

"VM Thread" prio=10 tid=0x000000000ae4e000 nid=0xab8 runnable 

"GC task thread#0 (ParallelGC)" prio=6 tid=0x000000000258d000 nid=0x1bb4 runnable 

"GC task thread#1 (ParallelGC)" prio=6 tid=0x000000000258e800 nid=0x1be4 runnable 

"GC task thread#2 (ParallelGC)" prio=6 tid=0x0000000002590800 nid=0x19f8 runnable 

"GC task thread#3 (ParallelGC)" prio=6 tid=0x0000000002592000 nid=0x1b60 runnable 

"VM Periodic Task Thread" prio=10 tid=0x000000000c29d800 nid=0x13a4 waiting on condition 

JNI global references: 107


Found one Java-level deadlock:
=============================
"哲学家2":
  waiting to lock monitor 0x000000000ae5d1e8 (object 0x00000007d5d9b028, a java.lang.Object),
  which is held by "哲学家1"
"哲学家1":
  waiting to lock monitor 0x000000000ae5bd48 (object 0x00000007d5d9b038, a java.lang.Object),
  which is held by "哲学家2"

Java stack information for the threads listed above:
===================================================
"哲学家2":
	at com.test.Test.run(Test.java:36)
	- waiting to lock <0x00000007d5d9b028> (a java.lang.Object)
	- locked <0x00000007d5d9b038> (a java.lang.Object)
"哲学家1":
	at com.test.Test.run(Test.java:24)
	- waiting to lock <0x00000007d5d9b038> (a java.lang.Object)
	- locked <0x00000007d5d9b028> (a java.lang.Object)

Found 1 deadlock.
```

很显然，我们成功检测到了死锁，并且详细列出了死锁的具体情况。