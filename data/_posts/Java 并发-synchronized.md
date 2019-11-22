---
title: Java 并发-synchronized
date: 2017-07-13 19:33:36
tags: [Java,并发,synchronized]
categories: Java 并发
---

在synchronized标记范围内的同步块同时只能被一个线程进入并执行操作，所有其他等待进入该同步块的线程将被阻塞，直到该同步块中的线程退出。

<!-- more -->

synchronized的含义为同步。在表现上，synchronized实现了监视器机制。

被阻塞的是尝试进入synchronized同步块的线程，如果说线程被阻塞相当于在前行的路上被一道门挡住了，则synchronized所修饰的对象(称为监视器对象)则相当于加于这道门上的监视器。在这里被当做监视器的对象仅仅相当于一个记号，所有对象都可被用作监视器，而被作为监视器对对象而言也没什么影响，受影响的仅仅是被阻塞了的线程。

synchronized相当于在线程前进的路上设置了两道门。进第一道门时需要获得监视器对象的认可，同一时间点最多只能有一个线程可以得到这个认可。若线程获得认可进入第一道门后，则可继续前行直至第二道门，第二道门没有监视器，其只相当于一个线程的退出标记，当线程离开第二道门后第一道门上的监视器对象将收回对该线程的认可，并可将该认可再次发给其他线程(当然只要刚离开第二道门的线程愿意，它也可以马上再次申请该认可)。

本质上，synchronized只能修饰对象，synchronized有如下具体的使用方式：

修饰实例方法。实际是修饰该实例方法所属的对象。

```
public synchronized void m() {}
```

修饰静态方法。实际是修饰该静态方法所属类的类对象(即Class对象)。

```
public static synchronized void m() {}
```

实例/静态 方法中的同步块。此时synchronized会显示指明监视器对象。

```
synchronized(o) {}
```

如果一个线程在等待监视器对象的认可，那么等待的结果只有两种：要么获得认可结束等待；要么无限期的等下去。换句话说，等待监视器对象认可是没有中断机制的。

synchronized同步块中的变量具有可见性。且并发线程执行到synchronized同步块时相当于变回了串行执行，因此synchronized天然具有有序性。由此synchronized可实现volatile的所有功能，只是付出的代价要大一些。

# 通信：wait()/wait(long timeout)，notify()/notifyAll()。

这几个方法都出自Object类：

```
public final void wait() throws InterruptedException {
    wait(0);
}

public final native void wait(long timeout) throws InterruptedException;

public final native void notify();

public final native void notifyAll();
```

wait()内部实际上就是调用wait(long timeout)，0表示无限期等待：

因此本质上二者的实现机制是一致的，wait()表示无限等待，只要没有接到唤醒信号就会一直等下去。wait(long timeout)除了会在接到唤醒信号时结束等待之外，也会在设置的限定时间到期后自动结束等待。除此之外二者没有其他差异。为了论述方便，下文仅以wait()来代指 wait()/wait(long timeout) 。

关于wait()，notify()和notifyAll()与Java线程状态转换的关系，详见[Java 并发-线程状态转换](/2017/07/12/Java 并发-线程状态转换/)。

为了进行线程间的通信，可采用"自定义通信对象+忙等待查询"的方式，详见[Java 并发-线程通信](/2017/07/13/Java 并发-线程通信/)。

忙等待的缺陷：从JVM的角度来看，JVM并不知道进行忙等待的线程是在做无法产生有效结果的等待操作，在它看来，该线程依然在正常的运行着并在切实执行用户程序。此时该线程并没有让出它已获得的所有资源。用户程序人为的让该线程在等待时适当的进行睡眠可以在一定程度上缓解这个问题，但是睡眠频度及睡眠时间的长短很难根据系统当前情况调整为一个最优值。

因此，Java提供了一个内建机制以从根本上解决这个问题，将用户程序从复杂的通信问题中解放出来。

java.lang.Object类定义了三个实例方法：wait()，notify()和notifyAll()来实现这个等待机制。注意该机制必须配合synchronized关键字使用，即这3个实例方法必须存在于synchronized标记范围内的同步块中，且该synchronized所监视的监视器对象即为这3个实例方法所属的对象(若不是，可通过编译，运行时抛出运行时异常IllegalMonitorStateException)。换句话说，要想使用某监视器对象的通信机制，必须要先获得该监视器对象的认可。因为同一时间点最多只能有一个线程可以得到这个认可，所以在同一时间点，对于某特定监视器对象而言，最多只有一个线程能调用这3个方法中的任意一个。

wait()：让出监视器对象的认可及CPU。因为唯有这样其他线程才有机会获得这个认可从而调用notify()或notifyAll()正常结束该线程的等待状态。该方法会throws受检查异常：InterruptedException。

notify()：随机唤醒监视器对象等待池中的一个线程。

notifyAll()：唤醒监视器对象等待池中的所有线程。

因为监视器对象的认可只有一个，因此notify()及notifyAll()只负责通知被唤醒的线程：你被唤醒了，可以不用等了。至于被唤醒的线程能否获得监视器对象的认可。这两个方法就不管了。具体来说，需要关注的问题有两个：

1. 和wait()不同，线程调用notify()或notifyAll()相当于只发出了一个通知，notify()或notifyAll()对其而言只是一个普通的方法，调用完成后线程不会让出任何资源，并且会继续向下执行。因为不让出任何资源，自然也不会让出其已获得的监视器对象的认可。因此不是调用notify()或notifyAll()后被唤醒线程立刻就能获得监视器对象的认可了，起码也要等调用线程让出该认可后才有可能。而只有重新获得了监视器对象的认可，被阻塞的线程才能够从等锁池中解放出来从而继续向下执行。

2. 基于同样的原理，notifyAll()虽然唤醒了所有线程，但是哪个线程能先得到监视器对象的认可，则要靠真本事去竞争了。不过这也只是先后问题，除非死锁或饿死，所有被唤醒的线程总有机会执行完成。

该通信机制有一个问题：notify()及notifyAll()是非常不负责任的方法，它们只是姑且通知一下，如果在通知的时候监视器对象的等待池中并没有在等待的线程。那么没有也就没有了，本次通知相当于作废。因此如果程序流程不当，导致notify()或notifyAll()调用在了其对应的wait()之前，这就会导致wait()永远无法再次接到唤醒信号了。

如下所示，该程序运行后无法结束。

```
public class Test {

    public static void main(String[] args) {
        Runnable wait = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000L);    // 设置情境：保证notify信号先发出
                    synchronized (Test.class) {
                        Test.class.wait();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }    
            }
        };
        Runnable notify = new Runnable() {
            @Override
            public void run() {
                synchronized (Test.class) {
                    Test.class.notify();
                }
            }
        };
        new Thread(wait).start();
        new Thread(notify).start();
    }
}
```

为解决该问题，可配合自定义通信对象一起使用：

```
public class Test {

    private static boolean NOTIFY_FLAG = false;

    private static final synchronized void setNOTIFY_FLAG(boolean nOTIFY_FLAG) {
        NOTIFY_FLAG = nOTIFY_FLAG;
    }

    public static void main(String[] args) {
        Runnable wait = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000L);    // 设置情境：保证notify信号先发出
                    if (!Test.NOTIFY_FLAG) {
                        synchronized (Test.class) {
                            Test.class.wait();
                        }
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Test.setNOTIFY_FLAG(false);    // 保证一个notify只为一个wait所用
            }
        };
        Runnable notify = new Runnable() {
            @Override
            public void run() {
                synchronized (Test.class) {
                    Test.setNOTIFY_FLAG(true);
                    Test.class.notify();
                }
            }
        };
        new Thread(wait).start();
        new Thread(notify).start();
    }
}
```

改造后，程序可执行完成。

# JVM中synchronized的实现方式

synchronized可以支持方法级的同步及方法内部一段指令序列的同步，在底层是通过管程(Monitor)来支持的。

方法级的同步是隐式的，即无需通过字节码指令来控制，它实现在方法调用和返回操作之中。JVM可通过判断class文件->方法表集合->方法表->access_flags中标记同步的标志ACC_SYNCHRONIZED是否为1来判断该方法是否是同步方法。若被设置为同步方法，执行线程需要先成功持有管程(即上文中的监视器许可)，然后才能执行方法，最后当执行完成(无论是正常完成还是非正常完成，即若一个同步方法在执行期间抛出了异常，且该方法无法处理这个异常，则执行线程将在该异常被抛出到同步方法之外时自动释放其所持有的管程)，都会释放管程。在其释放管程之前，其他任何线程都无法再取得这个管程。

而对于方法内部一段指令序列的同步，JVM提供了monitorenter及monitorexit指令标记同步区域。因此欲正确实现synchronized对方法内部一段指令序列的同步需javac编译器(将源码中被synchronized关键字修饰的代码块替换为对应的被monitorenter及monitorexit标记的字节码指令区域)及解释器(正确读取并执行monitorenter及monitorexit指令)二者共同支持。

例如有如下代码：

```
public class Test {

    public void m(Object o) {
        synchronized (o) {
            System.out.println("in synchronized...");
        }
    }
}
```

使用javap反编译其class文件：

```
Classfile /E:/Test.class
  Last modified 2017-11-22; size 531 bytes
  MD5 checksum e7d825a53a74620971ce4799ed6aad4a
  Compiled from "Test.java"
public class Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#19         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #20.#21        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #22            //  in synchronized...
   #4 = Methodref          #23.#24        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #25            //  Test
   #6 = Class              #26            //  java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               m
  #12 = Utf8               (Ljava/lang/Object;)V
  #13 = Utf8               StackMapTable
  #14 = Class              #25            //  Test
  #15 = Class              #26            //  java/lang/Object
  #16 = Class              #27            //  java/lang/Throwable
  #17 = Utf8               SourceFile
  #18 = Utf8               Test.java
  #19 = NameAndType        #7:#8          //  "<init>":()V
  #20 = Class              #28            //  java/lang/System
  #21 = NameAndType        #29:#30        //  out:Ljava/io/PrintStream;
  #22 = Utf8               in synchronized...
  #23 = Class              #31            //  java/io/PrintStream
  #24 = NameAndType        #32:#33        //  println:(Ljava/lang/String;)V
  #25 = Utf8               Test
  #26 = Utf8               java/lang/Object
  #27 = Utf8               java/lang/Throwable
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
{
  public Test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 1: 0

  public void m(java.lang.Object);
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=2    // 操作数栈最大深度为2
                                        // 局部变量表长度为4Slot，即支持的索引为[0,3]
					// 方法传入参数为2：this,java.lang.Object(后文记为o)
					// 初始时：
					// 局部变量表：this,o,无,无
					// 操作数栈(自栈底至栈顶)：无,无

         0: aload_1    // 将局部变量表1号Slot中的引用类型压入操作数栈
	               // 本指令完成后：
                       // 局部变量表：this,o,无,无
	               // 操作数栈(自栈底至栈顶)：o,无

         1: dup    // 弹出操作数栈栈顶数值，复制该值并将原值及复制值压入操作数栈
	           // 该复制值后文记为o_c
	           // 本指令完成后：
	           // 局部变量表：this,o,无,无
                   // 操作数栈(自栈底至栈顶)：o,o_c

         2: astore_2    // 将栈顶引用类型弹出并存入局部变量表2号索引处
	                // 本指令完成后：
			// 局部变量表：this,o,o_c,无
		        // 操作数栈(自栈底至栈顶)：o,无    

         3: monitorenter    // 弹出并获得操作数栈栈顶对象的管程，用于被synchronized关键字修饰的同步块
	                    // 本指令完成后：
			    // 局部变量表：this,o,o_c,无
			    // 操作数栈(自栈底至栈顶)：无,无

         4: getstatic     #2    // Field java/lang/System.out:Ljava/io/PrintStream;
	                        // 获取指定类的指定类变量，并将其值压入操作数栈。该指令的操作码之后会紧跟一个u2的操作数说明具体需要的是哪个类变量，该参数指向常量池集合中的一个CONSTANT_UTF8_info类型的索引项，也就是该字段的字段符号引用
			        // 本指令取得的类变量后文将记为out
			        // 本指令完成后：
				// 局部变量表：this,o,o_c,无
		                // 操作数栈(自栈底至栈顶)：out,无  

         7: ldc           #3    // String in synchronized...
	                        // 将int,float或String型常量值从常量池中推送至操作数栈栈顶。该指令的操作码之后会紧跟一个u2的操作数作为具体的值，该参数指向常量池集合中的一个对应类型的索引项
			        // 本指令对应的字符串后文中记为s
			        // 本指令完成后：
				// 局部变量表：this,o,o_c,无
				// 操作数栈(自栈底至栈顶)：out,s


         9: invokevirtual #4    // Method java/io/PrintStream.println:(Ljava/lang/String;)V
	                        // 调用实例方法。会根据对象的实际类型进行动态单分派(虚方法分派)
				// 弹出操作数栈中的out及s，out作为被调用方法的对象，s为方法所需的参数
			        // 本指令完成后：
				// 局部变量表：this,o,o_c,无
				// 操作数栈(自栈底至栈顶)：无,无

        12: aload_2    // 将局部变量表2号Slot中的引用类型压入操作数栈
		       // 本指令完成后：
		       // 局部变量表：this,o,o_c,无
		       // 操作数栈(自栈底至栈顶)：o_c,无

        13: monitorexit    // 弹出并释放操作数栈栈顶对象的管程，用于被synchronized关键字修饰的同步块
                           // 之所以用o_c而非o是为防止o被改变
			   // 本指令完成后：
		           // 局部变量表：this,o,o_c,无
			   // 操作数栈(自栈底至栈顶)：无,无

        14: goto          22    // 无条件跳转至指令22

        17: astore_3    // 将栈顶引用类型弹出并存入局部变量表3号索引处
	                // [4,13]发生异常后会跳转至本指令，不妨设12结束后发生异常。则操作数栈栈顶会被压入抛出的异常e
			// 本指令完成后：
			// 局部变量表：this,o,o_c,e
			// 操作数栈(自栈底至栈顶)：o_c,无
	
        18: aload_2    // 将局部变量表2号Slot中的引用类型压入操作数栈
		       // 本指令完成后：
		       // 局部变量表：this,o,o_c,e
		       // 操作数栈(自栈底至栈顶)：o_c,o_c
	
        19: monitorexit    // 弹出并释放操作数栈栈顶对象的管程，用于被synchronized关键字修饰的同步块
                           // 之所以用o_c而非o是为防止o被改变
			   // 本指令完成后：
		           // 局部变量表：this,o,o_c,e
			   // 操作数栈(自栈底至栈顶)：o_c,无

        20: aload_3    // 将局部变量表3号Slot中的引用类型压入操作数栈
		       // 本指令完成后：
		       // 局部变量表：this,o,o_c,e
		       // 操作数栈(自栈底至栈顶)：o_c,e

        21: athrow    // 弹出操作数栈栈顶的异常并将其抛出
	
        22: return    // 从当前方法返回void，本方法结束
	
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
      LineNumberTable:
        line 4: 0
        line 5: 4
        line 6: 12
        line 7: 22
      StackMapTable: number_of_entries = 2
           frame_type = 255 /* full_frame */
          offset_delta = 17
          locals = [ class Test, class java/lang/Object, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
           frame_type = 250 /* chop */
          offset_delta = 4

}

```

由上述反编译文件可证实，无论方法是否正常退出，只要执行了monitorenter(3)，就会执行对应的monitorexit(13,19，各可能的分支上都会冗余存储一份)。

# 一个隐蔽的错误加锁的示例

现有如下程序：

```
public class Test implements Runnable {

    private static Integer I = 0;

    @Override
    public void run() {
        for (int j = 0; j < 1000_0000; j++) {
            synchronized (Test.I) {
                I++;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Test r = new Test();
        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(Test.I);
    }
}
```

该程序实现了一个简单的计算器，乍看之下没有任何问题，预期运行结果为2000_0000。然而实际的运行结果为：

```
15429002
```

这比预期的结果小了很多，那么问题出在哪里呢？

在[Java 并发-Thread类](/2017/10/05/Java 并发-Thread类/)中，我们得出了这样一个结论：以Thread类的实例为监视器对象可能会引起未知的并发错误。可见监视器对象的错误选择是引起并发错误的一个重要原因，那么本例的错误是否也是由监视器对象I的选择错误引起的呢？

正是这样！I的类型为Integer:

```
public final class Integer extends Number implements Comparable<Integer>
```

这是一个被final修饰的不可变的类。也就是说一旦Integer的实例被创建，比如我们将其赋值为1，那么它终其一生就只能是1，不能变成其他值。而我们在程序中为什么又可以使用诸如I++来改变I的值呢？这其实是Java的一个小戏法：改变I的值相当于新建了一个Integer实例。

更具体的说，I++其实是这样实现的：

```
I = Integer.valueOf(I.intValue() + 1);
```

然后我们再来看valueOf()这个方法的源码：

```
public static Integer valueOf(int i) {
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

这样原因就明确了：监视器对象I所指向的实例相当于始终处于变化中，自然无法保证同步。由此看来，尽量不要选择不可变对象为监视器才好。

修改本例：

```
package com.test;

public class Test implements Runnable {

    private static Integer I = 0;

    @Override
    public void run() {
        for (int j = 0; j < 1000_0000; j++) {
            synchronized (Test.class) {
                I++;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Test r = new Test();
        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(Test.I);
    }
}
```

输出：

```
20000000
```

符合预期，没有发生并发问题。