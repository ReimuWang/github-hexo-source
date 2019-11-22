---
title: Java 并发-ThreadLocal
date: 2018-05-02 11:32:36
tags: [Java,并发,ThreadLocal]
categories: Java 并发
---

模拟一种场景：100个人需要填写个人信息表。有如下两种实施策略：

1. 只准备一支笔，这样做的好处是节省开销，坏处是同时只能有一个人在填表，效率低下，同时还必须花费时间和精力决策这支笔何时给何人使用。

2. 准备100支笔，即做到"人手一支笔"。这样做的好处就是每个人都可以并行的自己填自己的了，效率很高，同时也免去了对笔的调度问题。坏处就是100支笔较之一支笔开销剧增。

<!-- more -->

JVM中的锁及同步使用的就是第一种思路：即典型的用时间换空间(空间可理解为开销)。而本文欲介绍的ThreadLocal使用的就是第二种思路了：即用空间换时间。ThreadLocal就是"人手一支笔"的管理者。

稍微扩展一下，在并发环境中，通过用空间换时间的做法，将线程不安全的临界区变为线程安全的例子其实还有很多。例如，[JVM-堆中对象的创建及布局](/2017/10/18/JVM-堆中对象的创建及布局/)中所介绍的JVM为实例分配内存时用到的本地线程分配缓冲(Thread Local Allocation Buffer, TLAB)运用的就是这种思想。再比如，[Java 并发-读写锁ReadWriteLock](/2018/01/08/Java 并发-读写锁ReadWriteLock/)的实现思路也是类似的。

# ThreadLocal的简单使用

ThreadLocal在java.lang包中，其类定义为：

```
public class ThreadLocal<T>
```

我们先来看一段代码：

```
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {

    private static final DateFormat DF = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    private static class Parsedate implements Runnable {

        private int i;

        private Parsedate(int i) {
            this.i = i;
        }

        @Override
        public void run() {
            try {
                Date date = Test.DF.parse("1990-06-05 14:21:" + (this.i % 60));
                System.out.println(this.i + ":" + date);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) es.execute(new Parsedate(i));
	es.shutdown();
    }
}
```

执行后输出为：

```
	at com.test.Test$Parsedate.run(Test.java:25)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:745)
650:Tue Jun 05 14:21:50 CDT 1990
651:Tue Jun 05 14:21:51 CDT 1990
652:Tue Jun 05 14:21:52 CDT 1990
653:Mon Jun 05 14:21:53 CST 9690
655:Tue Jun 05 14:21:55 CDT 1990
656:Tue Jun 05 14:21:56 CDT 1990
Exception in thread "pool-1-thread-231" java.lang.NumberFormatException: empty String
	at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1020)
	at java.lang.Double.parseDouble(Double.java:540)
	at java.text.DigitList.getDouble(DigitList.java:168)
	at java.text.DecimalFormat.parse(DecimalFormat.java:1321)
	at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1793)
	at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1455)
	at java.text.DateFormat.parse(DateFormat.java:355)
	at com.test.Test$Parsedate.run(Test.java:25)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:745)
657:Tue Jun 05 14:21:57 CDT 1990
659:Thu May 31 14:21:59 CDT 1990
660:Tue Jun 05 14:21:00 CDT 1990
663:Tue Jun 05 14:21:03 CDT 1990
664:Tue Jun 05 14:21:04 CDT 1990
```

输出很长，在这里我们只截取其中一部分，因为这就足以说明问题了：很显然，程序获得了部分输出，却也抛出了异常，说明程序是有问题的。其问题就在于SimpleDateFormat的parse()方法不是线程安全的，将其置于并发环境中自然会产生问题。

为了解决这个问题，首先，我们可以使用思路一：即最常见的，使用锁或同步限制线程对临界区的访问：

```
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Test {

    private static final DateFormat DF = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    private static Lock LOCK = new ReentrantLock();

    private static class Parsedate implements Runnable {

        private int i;

        private Parsedate(int i) {
            this.i = i;
        }

        @Override
        public void run() {
            try {
                Test.LOCK.lock();
                Date date = Test.DF.parse("1990-06-05 14:21:" + (this.i % 60));
                Test.LOCK.unlock();
                System.out.println(this.i + ":" + date);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) es.execute(new Parsedate(i));
        es.shutdown();
    }
}
```

输出依然很长，不过不会有异常发生了。在此我们只截取最开始的那部分：

```
6:Tue Jun 05 14:21:06 CDT 1990
5:Tue Jun 05 14:21:05 CDT 1990
7:Tue Jun 05 14:21:07 CDT 1990
4:Tue Jun 05 14:21:04 CDT 1990
3:Tue Jun 05 14:21:03 CDT 1990
1:Tue Jun 05 14:21:01 CDT 1990
0:Tue Jun 05 14:21:00 CDT 1990
2:Tue Jun 05 14:21:02 CDT 1990
9:Tue Jun 05 14:21:09 CDT 1990
```

下面我们采取思路二，即使用ThreadLocal：

```
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {

    private static ThreadLocal<DateFormat> TL = new ThreadLocal<DateFormat>();

    private static class Parsedate implements Runnable {

        private int i;

        private Parsedate(int i) {
            this.i = i;
        }

        @Override
        public void run() {
            try {
                if (null == Test.TL.get())
                    Test.TL.set(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
                Date date = Test.TL.get().parse("1990-06-05 14:21:" + (this.i % 60));
                System.out.println(this.i + ":" + date);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) es.execute(new Parsedate(i));
        es.shutdown();
    }
}
```

输出同前文使用锁的代码时一样，就不再赘述了。

# ThreadLocal的实现原理

虽然使用ThreadLocal成功实现了功能。但是上节中ThreadLocal的Demo代码与我最初的设想却不大一样。我的想法是ThreadLocal既然是为每个线程分配的私有空间。那么应该以线程的成员变量的形式出现。然而它却是一个从属于类的静态变量，是唯一的。看来为每个线程划分空间的操作是在其内部实现的了。而其重点，自然就是set()及get()方法了。

咱们先来看set()方法的源码：

```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)    // 懒加载，使用前确认是否真的存在
        map.set(this, value);
    else
        createMap(t, value);
}
```

哈哈！代码并不长，而且正如我们所推测的，set()内部进行了线程私有空间的划分。显然，ThreadLocalMap类型的变量map正是这个空间。

那么我们具体来看看它到底是个啥吧。首先是getMap(t)的源码：

```
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

然后是createMap(t, value)的源码：

```
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

代码都很短，而且我们惊奇的发现，实际所使用的空间居然是线程本身的局部变量threadLocals：

```
ThreadLocal.ThreadLocalMap threadLocals = null;
```

ThreadLocalMap居然又是ThreadLocal的一个静态内部类：

```
static class ThreadLocalMap
```

因为ThreadLocal与Thread同在java.lang包中，所以这些类和变量都可自由的访问。

乍看一下，这关系可真够乱的。不过仔细梳理一下，其实还是很有条理的：既然是线程的私有空间，那么理应是存在于线程内部，这就是threadLocals。它的类型是ThreadLocalMap，其可以被理解为一个Map(虽然其实并没有实现Map接口，但是可以这样理解)。既然是Map，那么最核心的自然是key-value。它的key是ThreadLocal，而value则是实际需存储的变量。

依然是有点乱...

不过没关系，我们不妨举一个更具体的小例子：假设有3个ThreadLocal，我们不妨将其命名为t1,t2,t3。然后当前系统中并发工作的线程同样有3个。实际存储线程私有变量的容器是线程的threadLocals。如果需保存的私有变量就是字符串的话，那么某时刻系统的状态可能是这样的：

-线程1的threadLocals：{t1:"1-1",t2:"2-1",t3:"3-1"}
-线程2的threadLocals：{t1:"1-2",t2:"2-2",t3:"3-2"}
-线程3的threadLocals：{t1:"1-3",t2:"2-3",t3:"3-3"}

那么我们为什么会产生之前的误解呢？

我们回到文首，有这样一段话：

***JVM中的锁及同步使用的就是第一种思路：即典型的用时间换空间(空间可理解为开销)。而本文欲介绍的ThreadLocal使用的就是第二种思路了：即用空间换时间。ThreadLocal就是"人手一支笔"的管理者***

其中说ThreadLocal是"人手一支笔"的管理者，但是我(我想应该不仅仅是我，很多人都会这样想)却理所当然的将ThreadLocal看作是那支笔本身。但实际上ThreadLocal却是笔的记录者，并不是笔本身，笔还是在线程手中。

如果还是有些迷糊，我们不妨将上文中t1,t2,t3的小例子说得更具体一些。即继续扩展文章开头的那个填信息表的小例子：

-t1:笔的管理者
-t2:修改工具的管理者
-t3:纸张的管理者

线程1，2，3则分别对应于答卷人1，2，3。

那么在答卷过程中，某一时刻的资源分配状况可能是这样的：

-答卷人1：{拥有的笔:钢笔,拥有的修改工具:胶带,拥有的纸张:A4纸}
-答卷人2：{拥有的笔:铅笔,拥有的修改工具:修改液,拥有的纸张:B5纸}
-答卷人3：{拥有的笔:圆珠笔,拥有的修改工具:橡皮,拥有的纸张:A3纸}

而t1作为笔的管理者，其可以通过某种渠道查询到所有答卷人对笔的拥有情况。

所以简单来说，ThreadLocal可以被理解为某一类型资源的指针，该类型资源的实体还是存在于线程本身的内部。

然后，我们再来继续看get()方法的源码：

```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}
```

有了上文的分析，方法本身其实没什么好说的了。我们可以来看下setInitialValue()，它会在没有查到值时设置并返回一个默认值：

```
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

基本的代码逻辑与前文介绍的set()方法相似，只是设置的值为默认值，用到了initialValue()方法：

```
protected T initialValue() {
    return null;
}
```

直接返回一个null，简单粗暴。之所以封装为一个方法，推测是为了方便继承时重写，提高代码的灵活度。

通过上文的分析，我们知道了作为线程私有空间的threadLocals是Thread类的一个成员变量，那么我们很容易得出的一个结论就是：只要线程不退出(即Thread对象不被销毁)，threadLocals就不会被回收--因为Thread对象对它的引用将一直存在的。

我们先来说说线程退出时会发生什么。当线程欲退出时，系统会调用Thread对象的exit()方法：

```
private void exit() {
    if (group != null) {
        group.threadTerminated(this);
        group = null;
    }
    target = null;
    // 加速资源清理
    threadLocals = null;
    inheritableThreadLocals = null;
    inheritedAccessControlContext = null;
    blocker = null;
    uncaughtExceptionHandler = null;
}
```

很常见的套路，基本是将该Thread关联的所有引用都断掉了，其中就包括threadLocals。

而正如上文所分析的，若Thread对象不销毁，其所关联的threadLocals就不会被回收，那么这会导致什么问题呢？

通常来说，是不会有问题的。

但是Thread却并不是一个通常的类。说它不通常，倒不在这个类本身的代码编写上。而是在人的认识上：通常我们认为的线程被销毁，其实是任务被销毁了，而非线程本身。

举个例子，当我们在使用线程池时，线程池管理的是容器Thread，我们提交的Runnable则是其灵魂。任务完成，灵魂死去，但这并不意味着容器也一定被销毁了(可能被销毁，也可能没有，其结果是不受程序员控制的)。很有可能当下一个Runnable来的时候仍然是在复用上一个Thread。也就是说：我们认为Thead被销毁了，但是其实并没有。而只要Thread对象没有被真正的回收，它所关联的threadLocals就不会被回收，如果此时我们仍误以为上一个任务填入的ThreadLocal这个key已被销毁，就会导致threadLocals越来越大，有内存泄漏的风险。另外，如果读先发生于写的话，此时取出来的将不会是默认的空值，而是上一个任务留下的脏值(这里我们假定线程均共用一个ThreadLocal)。

因此，为了避免这种情况的发生，可以在任务完成时，手动显式的移除已没有用的变量。用到的方法为ThreadLocal中的remove()：

```
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

上述做法是建立在ThreadLocal对象还有用的前提下，如果我们确认该ThreadLocal对象已经没用了，或者我们愿意在下次使用前重新new出ThreadLocal对象的值，那其实可以采取更简单的策略：直接将该ThreadLocal对象置为null即可。

先不说"为什么"，咱们先来证明"是不是"：

```
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {

    private static volatile ThreadLocal<DateFormat> TL = new ThreadLocal<DateFormat>() {
        @Override
        protected void finalize() {
            System.out.println(this.toString() + " is gc");
        }
    };

    private static final int CD_LENGTH = 10000;

    private static volatile CountDownLatch CD = new CountDownLatch(Test.CD_LENGTH);

    private static class Parsedate implements Runnable {

        @SuppressWarnings("serial")
        @Override
        public void run() {
            try {
                if (null == Test.TL.get()) {
                    Test.TL.set(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss") {
                        @Override
                        protected void finalize() {
                            System.out.println(this.toString() + " is gc");
                        }
                    });
                    System.out.println(Thread.currentThread().getId() + " create SimpleDateFormat");
                }
            } finally {
                Test.CD.countDown();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < Test.CD_LENGTH; i++) es.execute(new Parsedate());
        Test.CD.await();
        System.out.println("first round complete");
        Test.TL = null;
        System.gc();
        System.out.println("first gc complete");
        es.shutdown();
    }
}
```

输出如下：

```
9 create SimpleDateFormat
13 create SimpleDateFormat
11 create SimpleDateFormat
18 create SimpleDateFormat
14 create SimpleDateFormat
16 create SimpleDateFormat
17 create SimpleDateFormat
12 create SimpleDateFormat
15 create SimpleDateFormat
10 create SimpleDateFormat
first round complete
first gc complete
com.test.Test$1@261c2628 is gc
```

com.test.Test$1@261c2628其实就是TL，不过因为是匿名内部类，因此看上去名字有些怪。

令我们感到遗憾的是，我们申请的那些SimpleDateFormat并没有被回收。不过不要急，我们对代码稍加修改，再跑一轮：

```
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {

    private static volatile ThreadLocal<DateFormat> TL = new ThreadLocal<DateFormat>() {
        @Override
        protected void finalize() {
            System.out.println(this.toString() + " is gc");
        }
    };

    private static final int CD_LENGTH = 10000;

    private static volatile CountDownLatch CD = new CountDownLatch(Test.CD_LENGTH);

    private static class Parsedate implements Runnable {

        @SuppressWarnings("serial")
        @Override
        public void run() {
            try {
                if (null == Test.TL.get()) {
                    Test.TL.set(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss") {
                        @Override
                        protected void finalize() {
                            System.out.println(this.toString() + " is gc");
                        }
                    });
                    System.out.println(Thread.currentThread().getId() + " create SimpleDateFormat");
                }
            } finally {
                Test.CD.countDown();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService es = Executors.newFixedThreadPool(10);
        for (int i = 0; i < Test.CD_LENGTH; i++) es.execute(new Parsedate());
        Test.CD.await();
        System.out.println("first round complete");
        Test.TL = null;
        System.gc();
        System.out.println("first gc complete");
        Test.TL = new ThreadLocal<DateFormat>();
        Test.CD = new CountDownLatch(Test.CD_LENGTH);
        for (int i = 0; i < Test.CD_LENGTH; i++) es.execute(new Parsedate());
        Test.CD.await();
        System.out.println("second round complete");
        System.gc();
        System.out.println("second gc complete");
        es.shutdown();
    }
}
```

输出如下：

```
18 create SimpleDateFormat
12 create SimpleDateFormat
10 create SimpleDateFormat
13 create SimpleDateFormat
11 create SimpleDateFormat
14 create SimpleDateFormat
16 create SimpleDateFormat
15 create SimpleDateFormat
17 create SimpleDateFormat
9 create SimpleDateFormat
first round complete
first gc complete
com.test.Test$1@5d3f57b3 is gc
15 create SimpleDateFormat
9 create SimpleDateFormat
17 create SimpleDateFormat
18 create SimpleDateFormat
10 create SimpleDateFormat
11 create SimpleDateFormat
16 create SimpleDateFormat
13 create SimpleDateFormat
12 create SimpleDateFormat
14 create SimpleDateFormat
second round complete
second gc complete
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
com.test.Test$Parsedate$1@4f76f1a0 is gc
```

很显然，这一次成了。那一串(正好10个)com.test.Test$Parsedate$1就是我们欲回收的第一轮的SimpleDateFormat。

虽然效果达到了，不过我们仍然有两个未解的问题：

1. 为什么第一轮不行，需要再跑一轮才能回收上一轮的资源呢？
2. 为什么居然能被回收？

我们先来解释下为什么第二个问题中要用"居然"这个词，也就是依常理来说下，为什么它们不该被回收。

我们在前文中说ThreadLocalMap类似于一个Map，那么我们不妨就以HashMap为例，来看看为什么不会被回收：

```
import java.util.HashMap;
import java.util.Map;

public class Test {

    public static void main(String[] args) {
        Map<Object, String> map = new HashMap<Object, String>();
        Object o1 = new Object();
        Object o2 = o1;
        map.put(o1, "v");
        o1 = null;
        System.out.println(map.get(o2));
    }
}
```

这是一个很适合作为面试笔试题的小程序，它的输出是：

```
v
```

o1指向一个Object实例，而后以o1为key将其放入map中，然后再将o1置为null。此时发生变化的仅仅只是o1指向的位置。对其原来指向的Object实例并没有影响。map中依然存在一个该Object实例的key，仅仅只是o1不指向它了而已。本程序中，为了能将该Object实例更容易的输出出来，还定义了另一个指向它的引用o2。事实上即便没有o2这个引用，map与Object实例依然是强引用关系。

因此，如果我们将ThreadLocalMap看作一个普通的Map的话，自然就会产生疑问：因为即便我们将作为key的ThreadLocal置为null，也仅仅是引用的变更，对ThreadLocal实例本身是没影响的，那么在ThreadLocalMap中作为该ThreadLocal实例value的SimpleDateFormat实例自然就不该被回收。

其原因就在于，正如前文所说的，ThreadLocalMap虽然可以被看作一个Map，但它却并非一个Map。硬要说的话，它更类似于WeakHashMap。ThreadLocalMap在其内部又定义了一个静态内部类Entry用以存储它所管理的数据：

```
static class Entry extends WeakReference<ThreadLocal> {
    Object value;

    Entry(ThreadLocal k, Object v) {
        super(k);
        value = v;
    }
}
```

因此，如果说普通的引用关系是强引用的话，ThreadLocalMap所采用的就是弱引用。

我们曾在[JVM-垃圾收集](/2017/10/26/JVM-垃圾收集/)中详细讨论过垃圾收集器对各种引用的处理策略。我们不妨将其再次摘录一遍：

从JDK1.2起，Java将引用的概念扩充为4种，强度从强至弱依次为：

1. 强引用(Strong Reference)：即为JDK1.1中的传统意义上的引用。程序中绝大多数的引用(诸如Object o = new Object())均是强引用。垃圾收集器绝不会收集通过强引用可达GC ROOT的对象。

2. 软引用(Soft Reference)：使用SoftReference类实现。该引用即为前文"例如"所描述的那种引用：当某次垃圾收集后内存依然不够用，会进行第二次垃圾收集，此次收集将无视软引用。

3. 弱引用(Weak Reference)：使用WeakReference类实现。也就是所谓的"消耗性引用"：经过一次垃圾收集后，该引用即失效。

4. 虚引用(Phantom Reference)：使用PhantomReference类实现。又名幽灵引用或幻影引用。该引用并不是一个真正的引用，也无法在可达性计算中发挥任何作用，其存在价值仅仅为对象被回收后能发出一个系统通知。

太棒了！

很显然，它完美的解决了我们的第二个疑问。不仅如此，它还解决了我们的第一个疑问：因为弱引用是消耗性引用，因此它至少还能坚持过一次gc。

# 引入ThreadLocal的意义

在均能实现功能的前提下，锁/同步 与ThreadLocal的选取还是要具体问题具体分析。通常来说，如果线程对临界区的争夺容易产生较大的性能损失，那么就更为推荐使用ThreadLocal了。一个典型的例子就是在并发环境下产生随机数。

话不多说，直接看代码吧：

```
import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class Test {

    /**
     * int, 每个线程产生的随机数个数
     */
    private static final int GEN_COUNT = 1000_0000;

    /**
     * long, 保证计算结果准确，两种计算方式所使用的Random的种子应相同
     * 随便定义一个值即可
     * 若不设定种子，种子将默认为Random的创建时间
     */
    private static final long SEED = 777L;

    /**
     * Random, 这是一个线程安全的Java类
     */
    private static final Random RANDOM = new Random(Test.SEED);

    private static ThreadLocal<Random> T_RANDOW = new ThreadLocal<Random>() {
        @Override
        protected Random initialValue() {
            // 统一赋初值，这样就不用每次都set了
            return new Random(Test.SEED);
        }
    };

    /**
     * 进行比对的任务类。
     * 有两种工作模式：
     * mode == 0: 使用锁
     * mode == 1: 使用ThreadLocal
     */
    private static class RandomTask implements Callable<Long> {

        private int mode;

        private RandomTask(int mode) {
            this.mode = mode;
        }

        @Override
        public Long call() throws Exception {
            long begin = System.currentTimeMillis();
            Random r = null;
            if (this.mode == 0)
                r = Test.RANDOM;
            else if (this.mode == 1)
                r = Test.T_RANDOW.get();
            for (int i = 0; i < Test.GEN_COUNT; i++) r.nextInt();
            long end = System.currentTimeMillis();
            long cost = end - begin;
            System.out.println(Thread.currentThread().getId() + " cost " + cost + "ms");
            return cost;
        }
        
    }

    private static void run(ExecutorService es, Future<Long>[] f, int model) throws InterruptedException, ExecutionException {
        for (int i = 0; i < f.length; i++) f[i] = es.submit(new RandomTask(model));
        long totalTime = 0L;
        for (int i = 0; i < f.length; i++) totalTime += f[i].get();
        String keyWord = null;
        if (model == 0)
            keyWord = "lock";
        else if (model == 1)
            keyWord = "ThreadLocal";
        System.out.println("use " + keyWord + " cost " + totalTime + "ms");
    }

    @SuppressWarnings("unchecked")
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        int threadCount = 10;    // 线程个数
        ExecutorService es = Executors.newFixedThreadPool(threadCount);
        Future<Long>[] f = new Future[threadCount];
        Test.run(es, f, 0);
        Test.run(es, f, 1);
        es.shutdown();
    }
}
```

输出：

```
11 cost 4263ms
17 cost 5531ms
15 cost 6006ms
16 cost 7025ms
14 cost 7944ms
10 cost 7993ms
12 cost 8021ms
18 cost 8073ms
9 cost 8095ms
13 cost 8092ms
use lock cost 71043ms
14 cost 298ms
15 cost 331ms
11 cost 340ms
12 cost 294ms
16 cost 367ms
10 cost 362ms
18 cost 329ms
9 cost 340ms
17 cost 434ms
13 cost 330ms
use ThreadLocal cost 3425ms
```

很显然，此时ThreadLocal的性能要远优于使用锁。