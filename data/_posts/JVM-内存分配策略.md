---
title: JVM-内存分配策略
date: 2017-11-02 17:03:49
tags: [Java,JVM]
categories: JVM
---

JVM的自动内存管理可归结为两部分：自动内存分配及自动内存回收。二者互为因果：因为分配了内存才会产生垃圾。而不进行垃圾清理就无法倒出空间用于分配。

从宏观的角度来看，分配和回收的都是对象，分配和回收的区域为堆(暂不考虑JIT优化导致的栈上分配的标量类型)。而主要的分配和回收的战场又可聚焦于Eden：对象生于Eden，绝大多数的对象也死在Eden。少数对象会进入Survivor，更少数的对象会进入老年代。这就是对象分配的基本规则，但是为了提高性能，JVM偶尔也会打破规则。

<!-- more -->

# 对象生于Eden

这是内存分配的基本规则。当Eden没有空间时会触发一次Minor GC。

示例代码如下：

```
public class Test {

    private static final int _1MB = 1024 * 1024;

    /**
     * -Xms20m -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:+UseSerialGC
     */
    public static void main(String[] args) {
        byte[] a1, a2, a3, a4;
        a1 = new byte[2 * Test._1MB];
        a2 = new byte[2 * Test._1MB];
        a3 = new byte[2 * Test._1MB];
        a4 = new byte[4 * Test._1MB];    // 触发Minor GC
    }
}
```

输出如下：

```
[GC[DefNew: 6816K->469K(9216K), 0.0055103 secs] 6816K->6613K(19456K), 0.0055674 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 4893K [0x00000000f9a00000, 0x00000000fa400000, 0x00000000fa400000)
  eden space 8192K,  54% used [0x00000000f9a00000, 0x00000000f9e51f90, 0x00000000fa200000)
  from space 1024K,  45% used [0x00000000fa300000, 0x00000000fa375608, 0x00000000fa400000)
  to   space 1024K,   0% used [0x00000000fa200000, 0x00000000fa200000, 0x00000000fa300000)
 tenured generation   total 10240K, used 6144K [0x00000000fa400000, 0x00000000fae00000, 0x00000000fae00000)
   the space 10240K,  60% used [0x00000000fa400000, 0x00000000faa00030, 0x00000000faa00200, 0x00000000fae00000)
 compacting perm gen  total 21248K, used 2570K [0x00000000fae00000, 0x00000000fc2c0000, 0x0000000100000000)
   the space 21248K,  12% used [0x00000000fae00000, 0x00000000fb082978, 0x00000000fb082a00, 0x00000000fc2c0000)
No shared spaces configured.
```

日志分析：

发生了一次Minor GC。欲分配a4的空间时的内存区域状况为：Eden使用6816K(a1+a2+a3)，Survivor及老年代均未使用。此时Eden的剩余空间已不足以分配a4所需的4MB空间。因此触发该次Minor GC。因单个Survivor的大小为1MB，不足以容纳a1/a2/a3中的任何一个，因此触发老年代的担保机制，a1,a2,a3均直接进入老年代，新生代大小由6816K减为469K。而因为确实也无法回收什么东西，所以堆总使用空间(6816K->6613K)几乎没有变化。本次GC后，Eden空出了足以分配a4的空间并将a4分配入其中。

进程结束前的内存使用情况：Eden使用8192*0.54=4424用于存储a4。老年代使用6144K用于存储a1,a2,a3。

# 大对象直接进入老年代

所谓的大对象即是指占用大量内存空间的对象。典型的大对象就是那种很长的字符串或数组。大对象对内存分配而言是灾难：举一个极端的例子，Eden大小为10MB，而程序不断产生大小为15MB的大对象，因为Eden无法分配支撑15MB对象的空间(Survivor一般会比Eden小得多，更没可能)，因此这些对象会直接进入老年代。更遭的是，若这些对象又是朝生夕死的，则又需高频度的触发Major GC。因此我们在写程序时应尽可能的避免写大对象，更不要写朝生夕死的大对象。

但也只能是尽量，JVM显然不能规定程序员不能写大对象，也不能在程序员写了大对象后就无法优化。另一个角度来说，到底多大的对象算是大也难于界定。

对于Parallel Scavenge而言，会自动进行优化，无需程序员多操心。而对于Serial及ParNew而言，提供了PretenureSizeThreshold来作为直接晋升老年代对象的大小阀值。即大于该值的对象将直接在老年代分配。该值单位默认为B且无法修改。即设置晋升阀值大小1MB(1024*1024=1048576)为-XX:PretenureSizeThreshold=1048576。设置该值后可避免Eden及Survivor之间发生不必要的大对象复制。

举一个小例子：

```
public class Test {

    private static final int _1MB = 1024 * 1024;

    /**
     * -Xms20m -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:+UseSerialGC -XX:PretenureSizeThreshold=1048576
     * 设定直接晋升老年代的对象大小阀值为1MB
     */
    public static void main(String[] args) {
        byte[] a = new byte[4 * Test._1MB];
    }
}
```

输出如下：

```
Heap
 def new generation   total 9216K, used 835K [0x00000000f9a00000, 0x00000000fa400000, 0x00000000fa400000)
  eden space 8192K,  10% used [0x00000000f9a00000, 0x00000000f9ad0fb8, 0x00000000fa200000)
  from space 1024K,   0% used [0x00000000fa200000, 0x00000000fa200000, 0x00000000fa300000)
  to   space 1024K,   0% used [0x00000000fa300000, 0x00000000fa300000, 0x00000000fa400000)
 tenured generation   total 10240K, used 4096K [0x00000000fa400000, 0x00000000fae00000, 0x00000000fae00000)
   the space 10240K,  40% used [0x00000000fa400000, 0x00000000fa800010, 0x00000000fa800200, 0x00000000fae00000)
 compacting perm gen  total 21248K, used 2570K [0x00000000fae00000, 0x00000000fc2c0000, 0x0000000100000000)
   the space 21248K,  12% used [0x00000000fae00000, 0x00000000fb082920, 0x00000000fb082a00, 0x00000000fc2c0000)
No shared spaces configured.
```

显然，尽管Eden的空间足够，a还是被直接分配进了老年代。

# 长寿的对象进入老年代

按照基本理论，老年代只是一个保人，即只在Eden+一个Survivor的内容无法完全在GC后存入另一个Survivor时才会介入。

为了减少复制量，我们会直接让大对象进入老年代。同理，对于那些长寿的对象，即便Survivor空间足够，我们也没必要反复复制它们：因为挺过越多次GC的对象挺过即将到来的GC的概率越大。

对象初始年龄为0，每经过一次GC则年龄加1。-XX:MaxTenuringThreshold=?用于指定多大年龄的对象有资格晋升入老年代。默认值为15。特别的，若该值设定为0，对象仍需先尝试进入Eden，但在Eden已满时不会进入Survivor而会直接进入老年代。

举个小例子：

```
public class Test {

    private static final int _1MB = 1024 * 1024;

    /**
     * -Xms20m -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:+UseSerialGC -XX:MaxTenuringThreshold=0
     */
    public static void main(String[] args) {
        byte[] a1 = new byte[Test._1MB / 4];
        byte[] a2 = new byte[4 * Test._1MB];
        byte[] a3 = new byte[4 * Test._1MB];    // 触发Minor GC
    }
}
```

输出如下：

```
[GC[DefNew: 5188K->0K(9216K), 0.0033932 secs] 5188K->4821K(19456K), 0.0034431 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4178K [0x00000000f9a00000, 0x00000000fa400000, 0x00000000fa400000)
  eden space 8192K,  51% used [0x00000000f9a00000, 0x00000000f9e14820, 0x00000000fa200000)
  from space 1024K,   0% used [0x00000000fa300000, 0x00000000fa300000, 0x00000000fa400000)
  to   space 1024K,   0% used [0x00000000fa200000, 0x00000000fa200000, 0x00000000fa300000)
 tenured generation   total 10240K, used 4821K [0x00000000fa400000, 0x00000000fae00000, 0x00000000fae00000)
   the space 10240K,  47% used [0x00000000fa400000, 0x00000000fa8b5660, 0x00000000fa8b5800, 0x00000000fae00000)
 compacting perm gen  total 21248K, used 2570K [0x00000000fae00000, 0x00000000fc2c0000, 0x0000000100000000)
   the space 21248K,  12% used [0x00000000fae00000, 0x00000000fb082960, 0x00000000fb082a00, 0x00000000fc2c0000)
No shared spaces configured.
```

分配a3前的内存分配状态：Eden占用5188K(a1+a2)，Survivor与老年代为空。此时Eden已不足以容纳a3，触发Minor GC。因晋升年龄的阀值为0，因此a1与a2全部进入老年代(即便Survivor可以容纳a1)。

程序退出时的内存分配状况：新生代使用4178K(a3)，Survivor为空，老年代使用4821K(a1+a2)。

对上例稍作修改，让-XX:MaxTenuringThreshold=1，则输出：

```
[GC[DefNew: 5024K->725K(9216K), 0.0041689 secs] 5024K->4821K(19456K), 0.0042235 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 5149K [0x00000000f9a00000, 0x00000000fa400000, 0x00000000fa400000)
  eden space 8192K,  54% used [0x00000000f9a00000, 0x00000000f9e51f90, 0x00000000fa200000)
  from space 1024K,  70% used [0x00000000fa300000, 0x00000000fa3b5618, 0x00000000fa400000)
  to   space 1024K,   0% used [0x00000000fa200000, 0x00000000fa200000, 0x00000000fa300000)
 tenured generation   total 10240K, used 4096K [0x00000000fa400000, 0x00000000fae00000, 0x00000000fae00000)
   the space 10240K,  40% used [0x00000000fa400000, 0x00000000fa800010, 0x00000000fa800200, 0x00000000fae00000)
 compacting perm gen  total 21248K, used 2570K [0x00000000fae00000, 0x00000000fc2c0000, 0x0000000100000000)
   the space 21248K,  12% used [0x00000000fae00000, 0x00000000fb082960, 0x00000000fb082a00, 0x00000000fc2c0000)
No shared spaces configured.
```

显然，a1在GC后进入了Survivor中。

# 动态对象年龄判定

若Survivor中相同年龄的对象大小总和大于单个Survivor大小总空间的一半，则年龄大于等于该值的对象都会晋升入老年代，而不必等待对象的年龄到达MaxTenuringThreshold。

# 空间分配担保

老年代的诞生目的是为年轻代作保，但是却没人为老年代作保，从而可能会导致担保失败(Handle Promotion Failure)。因此在进行内存分配时就有乐观与悲观之别。模拟一个情景：Eden已满，触发一次Minor GC。若当前老年代的可容空间能容纳新生代所有的对象，则老年代这个保人是绝对稳的，无论是乐观或悲观都可以进行GC。但是若无法完全容纳，则对于悲观的策略而言此次Minor GC就有可能失败，将直接转而触发Full GC清理全堆。对于乐观的策略而言，会继续检查老年代最大可用空间是否大于历次Minor GC晋升老年代大小的平均值，若小于，则即便是乐观的策略也无法再乐观下去了：将转而触发一次Full GC。若大于，尽管本次Minor GC有风险，依然会进行Minor GC，若不幸担保失败，则再触发一次Full GC。

开关-XX:+HandlePromotionFailure用于控制这个策略。开启为乐观策略，关闭为悲观策略。

在JDK1.6 Update 24之后，虽然-XX:+HandlePromotionFailure开关依然存在，但已然失效，系统默认为开关打开时的状态。