---
title: JVM-JDK自带JVM工具
date: 2017-11-03 10:00:49
tags: [Java,JVM]
categories: JVM
---

解决Java程序JVM相关问题时，知识与经验是基础，数据是依据，工具是运用知识与经验处理数据的手段。

JVM相关的数据举例：

- 运行日志
- 异常堆栈
- GC日志
- 线程快照(threaddump/javacore)
- 堆转储快照(heapdump/hprof)

<!-- more -->

本文所介绍的环境为Windows平台下的JDK1.7.0_80，其bin目录中的内容如下：

![0.jpg](/images/blog_pic/JVM/JDK自带JVM工具/0.jpg)

这些工具大多只有十几K，之所以能做到如此小巧，是因为它们基本都是对jdk/lib/tools.jar的二次封装，可以对比看一下tools.jar解压后的内容：

![1.jpg](/images/blog_pic/JVM/JDK自带JVM工具/1.jpg)

![2.jpg](/images/blog_pic/JVM/JDK自带JVM工具/2.jpg)

# 命令行工具

**jps**

jps(JVM Process Status Tool)类似于Linux的ps命令，用于显示系统中所有的Hotspot VM进程，并显示JVM执行的主类(Main Class，即main()所在的类)及这些JVM进程的本地虚拟机唯一ID(Local Virtual Machine Identifier,LVMID)。LVMID是整个JDK监控工具体系的线索，用于作为后续监控工作的查询条件。

对于本地虚拟机进程而言，LVMID与操作系统的进程ID(Process Identifier,PID)是一致的。Windows下利用资源管理器Linux下利用ps均可得到该PID，但此时无法根据JVM执行的主类进行定位。

运行一个测试用的小程序:

```
public class Test {

    /**
     * -Xms20m -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:+UseSerialGC -XX:MaxTenuringThreshold=15
     */
    public static void main(String[] args) {
        while (true) ;
    }
}
```

将该程序运行起来后：

![3.jpg](/images/blog_pic/JVM/JDK自带JVM工具/3.jpg)

-q只输出LVMID。

![4.jpg](/images/blog_pic/JVM/JDK自带JVM工具/4.jpg)

-m输出启动时传递给main()的参数。

![5.jpg](/images/blog_pic/JVM/JDK自带JVM工具/5.jpg)

-l输出主类的全路径名。若进程执行的是Jar包则输出Jar路径。

![6.jpg](/images/blog_pic/JVM/JDK自带JVM工具/6.jpg)

-v输出JVM参数

![7.jpg](/images/blog_pic/JVM/JDK自带JVM工具/7.jpg)

**jstat**

jstat(JVM Statistics Monitoring tool)可用于监控JVM的运行时状态信息。它可以显示本地或远程JVM进程中的类装载，内存，垃圾收集，JIT编译等运行时数据。

```
public class Test {

    /**
     * -Xms20m -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:+UseSerialGC -XX:MaxTenuringThreshold=15
     */
    public static void main(String[] args) {
        while (true) ;
    }
}
```

![8.jpg](/images/blog_pic/JVM/JDK自带JVM工具/8.jpg)

6372是LVMID，1000是输出间隔(单位ms)，5是输出次数。

-gcutil：监控堆及方法区的内存使用情况，GC时间等信息。并重点关注已使用空间占总空间百分比。即各内存区域下的数值为已用空间占总空间的百分比。

- S0:Survivor0
- S1:Survivor1
- E:Eden
- O:old(老年代)
- P:Permanent(永久代)
- YGC:Young GC(Minor GC)次数
- YGCT:Young GC(Minor GC)耗时。单位为秒。
- FGC:Full GC次数
- FGCT:Full GC耗时，单位为秒。
- GCT:所有GC总耗时，单位为秒。

**jinfo**

jinfo(Configuration Info For Java)可查看JVM参数。jps -v可查看启动时显式指定的JVM参数，而jinfo能查询到包含默认值在内的所有参数。

```
public class Test {

    /**
     * -Xms20m -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:+UseSerialGC -XX:MaxTenuringThreshold=15
     */
    public static void main(String[] args) {
        while (true) ;
    }
}
```

![9.jpg](/images/blog_pic/JVM/JDK自带JVM工具/9.jpg)

即老年代与年轻代的比例默认为2:1。

**jmap**

jmap(Memory Map for Java)用于生成堆转储快照(heapdump/dump)。

```
public class Test {

    /**
     * -Xms20m -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:+UseSerialGC -XX:MaxTenuringThreshold=15
     */
    public static void main(String[] args) {
        while (true) ;
    }
}
```

![10.jpg](/images/blog_pic/JVM/JDK自带JVM工具/10.jpg)

该命令即在e盘下生成dump文件dumpTest。

**jhat**

由jmap dump出的堆转储快照无法直接阅读，需使用jhat(JVM Heap Analysis Tool)读取。jhat内置了一个微型的HTTP/HTML服务器用于分析阅读堆转储快照。

分析上小节中jmap产生的dumpTest：

![11.jpg](/images/blog_pic/JVM/JDK自带JVM工具/11.jpg)

![12.jpg](/images/blog_pic/JVM/JDK自带JVM工具/12.jpg)

分析内存泄漏问题时主要会用到其中的Show heap histogram，点进去：

![13.jpg](/images/blog_pic/JVM/JDK自带JVM工具/13.jpg)

该页面可显示进程中所有的同类实例数及总大小。

**jstack**

jstack(Stack Trace for Java)用于生成JVM当前时刻的线程快照(threaddump,javacore)。

```
public class Test {

    public static void main(String[] args) {
        Thread thread = new Thread() {
            @Override
            public void run() {
                synchronized (Test.class) {
                    try {
                        Test.class.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        thread.setName("testThread");
        thread.start();
    }
}
```

执行：

```
./jstack -l 7432 > /e/1.txt
```

其中7432为LVMID。输出文件1.txt中的内容为：

```
2017-11-03 16:35:11
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.80-b11 mixed mode):

"DestroyJavaVM" prio=6 tid=0x00000000024ce800 nid=0x1df4 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"testThread" prio=6 tid=0x000000000c3bc000 nid=0x1d34 in Object.wait() [0x000000000cf7e000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d5d9af10> (a java.lang.Class for com.test.Test)
	at java.lang.Object.wait(Object.java:503)
	at com.test.Test$1.run(Test.java:11)
	- locked <0x00000007d5d9af10> (a java.lang.Class for com.test.Test)

   Locked ownable synchronizers:
	- None

"Service Thread" daemon prio=6 tid=0x000000000acaf800 nid=0x1e8c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread1" daemon prio=10 tid=0x000000000acac800 nid=0xcbc waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread0" daemon prio=10 tid=0x000000000acab000 nid=0x1fa4 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Attach Listener" daemon prio=10 tid=0x000000000aca8000 nid=0x1df8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Signal Dispatcher" daemon prio=10 tid=0x000000000aca7000 nid=0x1b48 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Finalizer" daemon prio=8 tid=0x000000000ac2f000 nid=0x1b28 in Object.wait() [0x000000000c25e000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d5d04858> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	- locked <0x00000007d5d04858> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

   Locked ownable synchronizers:
	- None

"Reference Handler" daemon prio=10 tid=0x000000000ac2d800 nid=0x884 in Object.wait() [0x000000000c00e000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d5d04470> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:503)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	- locked <0x00000007d5d04470> (a java.lang.ref.Reference$Lock)

   Locked ownable synchronizers:
	- None

"VM Thread" prio=10 tid=0x000000000ac2a000 nid=0x187c runnable 

"GC task thread#0 (ParallelGC)" prio=6 tid=0x000000000251d000 nid=0x9b4 runnable 

"GC task thread#1 (ParallelGC)" prio=6 tid=0x000000000251e800 nid=0x1ffc runnable 

"GC task thread#2 (ParallelGC)" prio=6 tid=0x0000000002520800 nid=0x1ce4 runnable 

"GC task thread#3 (ParallelGC)" prio=6 tid=0x0000000002522000 nid=0x1ce8 runnable 

"VM Periodic Task Thread" prio=10 tid=0x000000000c3b3000 nid=0x1e34 waiting on condition 

JNI global references: 107
```

其中：

```
"testThread" prio=6 tid=0x000000000c3bc000 nid=0x1d34 in Object.wait() [0x000000000cf7e000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007d5d9af10> (a java.lang.Class for com.test.Test)
	at java.lang.Object.wait(Object.java:503)
	at com.test.Test$1.run(Test.java:11)
	- locked <0x00000007d5d9af10> (a java.lang.Class for com.test.Test)
```

表明名为testThread的线程wait在了com.test.Test的类对象上。

使用Thread.getAllStackTraces()可通过编程实现jstack的绝大多数功能。例如：

```
import java.util.Map;

public class Test {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread() {
            @Override
            public void run() {
                synchronized (Test.class) {
                    try {
                        Test.class.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        thread.setName("testThread");
        thread.start();
        Thread.sleep(100);    // 确保testThread已wait
        for (Map.Entry<Thread, StackTraceElement[]> stackTrace : Thread.getAllStackTraces().entrySet()) {
            System.out.println("=======" + stackTrace.getKey().getId() + "-" + stackTrace.getKey().getName());
            for (StackTraceElement element : stackTrace.getValue()) System.out.println(element);
        }
    }
}
```

输出如下：

```
=======4-Signal Dispatcher
=======3-Finalizer
java.lang.Object.wait(Native Method)
java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)
=======9-testThread
java.lang.Object.wait(Native Method)
java.lang.Object.wait(Object.java:503)
com.test.Test$1.run(Test.java:13)
=======1-main
java.lang.Thread.dumpThreads(Native Method)
java.lang.Thread.getAllStackTraces(Thread.java:1640)
com.test.Test.main(Test.java:23)
=======2-Reference Handler
java.lang.Object.wait(Native Method)
java.lang.Object.wait(Object.java:503)
java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
=======5-Attach Listener
```

# 可视化工具

**Jconsole**

Jconsole(Java Monitoring and Management Console)诞生于JDK1.5。是一种基于JMX(Java Management Extensions，即Java管理扩展)的可视化监视及管理工具。工具位置：jdk/bin/jconsole.exe。

示例代码：

```
public class Test {

    public static void main(String[] args) {
        Thread thread = new Thread() {
            @Override
            public void run() {
                synchronized (Test.class) {
                    try {
                        Test.class.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        thread.setName("testThread");
        thread.start();
        while (true) ;
    }
}
```

![14.jpg](/images/blog_pic/JVM/JDK自带JVM工具/14.jpg)

![15.jpg](/images/blog_pic/JVM/JDK自带JVM工具/15.jpg)

再来构造一个死锁的例子：

```
public class Test {

    public static void main(String[] args) {
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                synchronized ("八云紫") {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized ("八云蓝") {
                    }
                }
            }
        };
        Runnable r2 = new Runnable() {
            @Override
            public void run() {
                synchronized ("八云蓝") {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized ("八云紫") {
                    }
                }
            }
        };
        new Thread(r1, "r1").start();
        new Thread(r2, "r2").start();
    }
}
```

点击“检测死锁”：

![16.jpg](/images/blog_pic/JVM/JDK自带JVM工具/16.jpg)

随后多出了一个“死锁”标签。

r1：

![17.jpg](/images/blog_pic/JVM/JDK自带JVM工具/17.jpg)

r2：

![18.jpg](/images/blog_pic/JVM/JDK自带JVM工具/18.jpg)

**VisualVM**

VisualVM(All-in-One Java Troubleshooting Tool)诞生于JDK1.6 Update7。现已成为Sun主力推动的多合一故障处理，性能分析(Profiling)工具，并已从JDK中分离出来成为可以独立发展的开源项目。不需要被监视的程序基于特殊Agent运行，因此它对被监视的程序的实际影响很小。

程序位置为/jdk/bin/jvisualvm.exe。

构建一个死锁的例子：

```
public class Test {

    public static void main(String[] args) {
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                synchronized ("八云紫") {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized ("八云蓝") {
                    }
                }
            }
        };
        Runnable r2 = new Runnable() {
            @Override
            public void run() {
                synchronized ("八云蓝") {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized ("八云紫") {
                    }
                }
            }
        };
        new Thread(r1, "r1").start();
        new Thread(r2, "r2").start();
        while (true) ;
    }
}
```

有两种方式生成堆转储快照：

方法1：

![19.jpg](/images/blog_pic/JVM/JDK自带JVM工具/19.jpg)

方法2：

![20.jpg](/images/blog_pic/JVM/JDK自带JVM工具/20.jpg)

生成后：

![21.jpg](/images/blog_pic/JVM/JDK自带JVM工具/21.jpg)

![22.jpg](/images/blog_pic/JVM/JDK自带JVM工具/22.jpg)

实例信息需要由类信息点击进入：

![23.jpg](/images/blog_pic/JVM/JDK自带JVM工具/23.jpg)

生成的堆转储文件会在VisualVM关闭时删除，若欲保存生成的堆转储快照：

![24.jpg](/images/blog_pic/JVM/JDK自带JVM工具/24.jpg)

装入已存在的快照：

![25.jpg](/images/blog_pic/JVM/JDK自带JVM工具/25.jpg)