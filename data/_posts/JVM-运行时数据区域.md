---
title: JVM-运行时数据区域
date: 2017-10-17 16:44:49
tags: [Java,JVM]
categories: JVM
---

JVM的运行时数据区域即是指JVM所管理的内存区域。JVM规范依功能又将其分为如下几部分：

- 程序计数器(Program Counter Register)
- 虚拟机栈(Virtual Machine Stack)
- 本地方法栈(Native Method Stack)
- 堆(Heap)
- 方法区(Method Area)

JVM规范中详细指明了每条虚拟机指令的执行过程，执行前后对操作数栈及局部变量表的影响。Sun最早的JVM Sun Classic VM基本严格遵循这一规范。然而随着技术的发展及对性能的追求，高性能JVM真正的实现细节已与JVM规范产生了很大的差异。虽然"能做到什么"依然遵循JVM规范，然而"如何做到的"已不完全遵循规范的指引。

基于上述原因，再加之规范中本来就会有一些供实现自行发挥的弹性部分，因此基于同一个JVM规范得到的不同实现之间会有着不小的差异。若非特别指明，本文所讨论的实现均为Sun公司的HotSpot。

<!-- more -->

# 程序计数器

这是一块很小的，线程私有的内存区域，其生命周期与拥有其的线程相同。可看作是线程所执行字节码的行号指示器，用于记录线程正在执行的字节码的行号(实际上是偏移量)。

若某线程正在执行的字节码属于一个Java方法，则该线程的程序计数器中存储的是该字节码指令的行号。若某线程正在执行的字节码属于一个本地方法，此时该线程的程序计数器中的值为Undefined。

因为存储的数据简单明确，程序计数器是运行时数据区域中唯一不会抛出内存相关Error的区域。

# 虚拟机栈

虚拟机栈是线程私有的，它的生命周期与拥有其的线程相同。

每当线程调用Java方法，就会产生一个栈帧(Stack Frame)压入其自身的虚拟机栈，方法执行结束后对应的栈帧出栈。栈帧中记录了该方法的信息：

- 局部变量表
- 操作数栈
- 动态链接(记录符号引用所对应的实际地址，编译期不可知，运行期生成)
- 方法出口

等。

局部变量表中存放了其所属方法用到的：

- 基本数据类型
- 对象引用
- returnAddress: 指向一条字节码指令的地址，用于在代码发生跳跃并局部运行完成后指定返回的位置。例如try后跳入finally执行，finally执行完成后returnAddress会指示需要返回到的字节码指令的地址，JDK1.7中已取消returnAddress，改以冗余复制的方式完成try-catch-finally的功能

其中64位的long和double会占用两个Slot(局部变量表的基本单位)，其余则占用1个Slot。

操作数栈是和执行器关系最为密切的数据存储单元，执行器只能看到操作数栈中的数据。而操作数栈所需的数据则来源于局部变量表。操作数栈与局部变量表的关系有些类似于CPU的寄存器与内存(在此我们不妨将缓存视为内存的一部分)。例如，某方法欲计算1+2+4，那么可能的执行顺序是这样的：

1. 局部变量表中存入1,2,4
2. 从局部变量表中取出1与2压入操作数栈进行加法计算得到结果3
3. 将计算得到的3存入局部变量表
4. 从局部变量表中取出3与4压入操作数栈进行加法计算得到结果7
5. 将7存入局部变量表

全程执行器可见的只有操作数栈，而操作数栈中的值也必须通过存入局部变量表才能传递到外界。

局部变量表的大小及操作数栈的深度在编译期(.java --> .class)即可完全确定，运行期不会发生变化。

有人会将运行时数据区域比较粗旷的分为堆(Heap)及栈(Stack)。这种划分去掉了他们认为"不那么重要的"程序计数器及方法区。堆是我们通常认识的那个堆，而栈则去掉了本地方法栈仅仅指虚拟机栈。换句话说，这种划分认为Java运行时数据区域最重要的是如下两部分：堆及虚拟机栈。进一步的，更有人将虚拟机栈中的"不那么重要的"部分也去掉了，具体特指局部变量表。

虚拟机栈可能抛出的和内存相关的Error有两种：

- 压入的栈帧数超过了虚拟机栈允许的最大深度，将抛出StackOverflowError。
- 扩展栈(虚拟机栈默认即为可扩展的，当然也允许设置虚拟机栈为固定长度)或创建新栈时无法申请到足够的内存空间，则抛出OutOfMemoryError。

**OOM小例子**

因为HotSpot将虚拟机栈及本地方法栈合二为一，因此虽然-Xoss(设置本地方法栈大小)依然存在，但实际是无效的。栈容量只由-Xss控制。

栈可能会报的异常有两种：OutOfMemoryError，StackOverflowError。但从一个更大的角度来看，这两个错误描述的都是同一件事：无法分配新的栈空间了。至于具体原因，可能是因为空间不够了，也可能是因为压入的栈帧太多了。

为了使错误更容易出现，即更容易的促成"无法分配新的栈空间"。可行的方法有两种。

第一种是减少每个栈的总容量：

```
public class Test {

    private static int STACK_LENGTH;

    private static void stackTest() {
        Test.STACK_LENGTH++;
        Test.stackTest();
    }

    /**
     * -Xss256k
     */
    public static void main(String[] args) {
        try {
            Test.stackTest();
        } catch (Error e) {
            System.out.println("stack length=" + Test.STACK_LENGTH);
            throw e;
        }
    }
}
```

其中设置的-Xss256k是一个相对靠谱的正常的栈大小，运行后输出如下：

```
stack length=2728
Exception in thread "main" java.lang.StackOverflowError
	at com.test.Test.stackTest(Test.java:8)
	at com.test.Test.stackTest(Test.java:9)
	at com.test.Test.stackTest(Test.java:9)
	at com.test.Test.stackTest(Test.java:9)
	...
```

若改为-Xss8k，则输出：

```
The stack size specified is too small, Specify at least 104k
```

很显然，如果过于小的话那么是连启动都启动不起来的。于是改为-Xss65k(比错误信息里的最小值还要小不少，这是我的环境下试出的最小值，可见那个描述只是推荐值，实际没那么精确)，输出：

```
Exception in thread "main" stack length=1097
java.lang.StackOverflowError
	at com.test.Test.stackTest(Test.java:8)
	at com.test.Test.stackTest(Test.java:9)
	at com.test.Test.stackTest(Test.java:9)
	at com.test.Test.stackTest(Test.java:9)
	...
```

此时能触及的最大深度显然就小很多了。

第二种是增大每个栈帧的大小，最简便的方法就是疯狂的创建基本类型的局部变量(创建引用类型先爆的八成会是堆)，撑爆栈空间：尚未找到有效的测试代码，因为最先爆掉的总是堆。不过根据[深入理解Java虚拟机第二版]的说法，最终抛出的错误也是StackOverflowError(姑且先信了他，不知道他是怎么测的)。

无论如何，在单线程环境下都只能模拟出StackOverflowError的异常(尝试了将-Xss设置为一个极大的值，但貌似没有生效)。可以这样理解：例如-Xss256k，那么系统只要足以分配这256k的空间，超过该值抛出的错误都是StackOverflowError。而只有当系统连这最大的256k都无法分配的时候才会抛出OutOfMemoryError。因此自然想到通过创建多线程来触发这个Error：

```
public class Test {

    /**
     * -Xss3m
     */
    public static void main(String[] args) {
        while (true) {
            new Thread() {
                @Override
                public void run() {
                    while (true) ;
                }
            }.start();
        }
    }
}
```

因为在Windows平台的JVM中，Java线程是映射到操作系统的内核线程上的。因此执行上述代码时我的电脑壮烈的死机了。强制关机并重启才恢复了正常。因此自然也没有得到输出。不过同样是根据[深入理解Java虚拟机第二版]的说法，这段代码可以得到如下输出：

```
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
```

# 本地方法栈

本地方法栈与虚拟机栈的功能非常类似：同样是线程私有的，同样会抛出StackOverflowError及OutOfMemoryError。虚拟机栈为JVM执行Java方法(即字节码)服务，本地方法栈为JVM使用本地方法服务。

HotSpot VM将虚拟机栈与本地方法栈合二为一。

# 堆

该区域的唯一目的就是存储对象实例，JVM规范对其的描述为："所有的对象实例及数组都要在堆上分配内存"。但是在实际实现中，随着JIT编译器的发展，逃逸分析技术的日渐成熟，诸如栈上分配，标量替换等优化措施使得这个描述不那么准确了。不过我们仍可说，在具体实现中，"绝大多数的对象实例及数组都是在堆上分配内存的"。

对绝大多数应用而言，堆都是占用内存最大的一块区域。堆随着JVM的启动而创建，生命周期与JVM相同。很显然，堆中的对象实例是被所有线程所共享的。

堆是垃圾收集器收集垃圾的主要区域，因此堆有时也被称为GC堆(Garbage Collected Heap)(幸亏没直译为垃圾堆=-=)。

根据JVM规范中的规定，类似于磁盘空间，堆可以处于物理上不连续的内存空间中，只要逻辑上连续即可。具体实现时，既可以是固定大小的，也可以是可扩展的。不过当前主流的JVM实现都是可扩展的(Hotspot通过-Xmx及-Xms控制。-Xmx为最大堆空间，-Xms为最小堆空间)。

若当前堆无法满足分配需求且无法扩展，则抛出OutOfMemoryError。

**OOM小例子**

```
import java.util.ArrayList;
import java.util.List;

public class Test {

    /**
     * -Xms20m -Xmx20m
     */
    public static void main(String[] args) {
        List<Test> list = new ArrayList<Test>();
        while (true) list.add(new Test());
    }
}
```

执行后输出：

```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:2245)
	at java.util.Arrays.copyOf(Arrays.java:2219)
	at java.util.ArrayList.grow(ArrayList.java:242)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:216)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:208)
	at java.util.ArrayList.add(ArrayList.java:440)
	at com.test.Test.main(Test.java:13)
```

# 方法区

JVM规范将方法区描述为堆的一个逻辑组成部分，但是方法区与堆中所存储的对象实例又有明显区别。简单来说，只要能从具体实例中抽象出的属于类的模版信息都应存储在方法区中。

说方法区是堆的逻辑组成部分，是因为堆和方法区中描述的都是对象信息。方法区中存储的是干细胞，堆中存储的则是由干细胞产生的具体的细胞实例。但方法区与堆中存储的数据又存在明显的界限，因此它才会在分类时被完全并列出来单独列举，并且它还有一个别名：非堆(Non-Heap)。

特别的，用于描述类信息的，与每个类一一对应的Class类的实例，也被称为该类的类对象，作为一个普通的实例对象也是存在于堆中的，只不过其所封装的信息都来源于方法区(好像句柄呀)。

很显然，方法区中的数据也是线程共享的。和堆相同，方法区也可以处于物理上不连续的内存空间中，只要逻辑上连续即可。具体实现时，既可以是固定大小的，也可以是可扩展的。

在JDK1.6及此前的版本，HotSpot VM的方法区大致可分为如下两部分：

- 永久代：类的模版信息，例如类的描述信息，常量，静态变量。粗略的看，可认为是.class文件读入内存后存放在了永久代。所谓的对方法区的垃圾回收实际管理的就是这个区域，主要回收的内存为运行时常量池及类的卸载。
- 代码缓存：即时编译器(JIT)编译后的代码。

JVM规范中并未明确规定方法区该如何实现，也未规定垃圾收集器是否该收集这个区域的内容。具体到HotSpot VM，其方法区是以永久代(Permanent Generation)实现的，并将垃圾收集器的垃圾收集范围扩展到方法区。这样做的好处是垃圾收集器可以像管理堆一样来管理方法区，而不用特地为方法区去编写内存管理代码。但这样做却更容易发生内存溢出：永久代有-XX:MaxPermSize的上限，达到该上限后即便尚有内存空间也无法再行分配了。

因此，HotSpot VM在逐步废弃永久代。在JDK1.7中，储存在永久代中的部分数据就已经转移到了堆(例如字符串常量池，类的静态变量)或本地内存(例如符号引用)中。但永久代依然存在，并未完全移除。

我们不妨以String.intern()这个方法来证明该变化。该方法的作用为返回该字符串所对应的字符串常量池中的那个字符串的引用。具体来说，当我们调用：

```
str.intern()
```

会先检查str所对应的值是否已在字符串常量池中存在，若已存在则返回那个已存在的字符串值对应的对象引用。若不存在，JDK1.6时，字符串常量池尚在方法区中，因此需要以str的值为基础在方法区的字符串常量池中创建一个新的字符串对象，而后把这个对象的引用返回回去；而JDK1.7时字符串常量池本就在堆中，因此只需将该str对象的引用加入到字符串常量池中即可。

首先验证的就是字符串常量池的位置：

```
import java.util.ArrayList;
import java.util.List;

public class Test {

    /**
     * -XX:PermSize=10m -XX:MaxPermSize=10m
     */
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        int i = 0;
        while (true) list.add(String.valueOf(i++).intern());
    }
}
```

在JDK1.6中，运行结果为：

```
Exception in thread "main" java.lang.OutOfMemoryError:PermGen space
...
```

而在JDK1.7中，除非触碰到堆内存的分配上限(即便真的触碰上限抛出了Error，Error的区域也是Heap space而非PermGen space)，否则代码会一直运行下去。

其次验证的是String.intern()的功能：

```
public class Test {

    public static void main(String[] args) {
        String str = new String("紫婆婆");
        System.out.println(str.intern() == str);
    }
}
```

此时因为在创建str对象时代表"紫婆婆"这个字符串常量的字符串已然创建过并将其作为该常量值的引用加入到了字符串常量池，因此无论是JDK1.6抑或是JDK1.7，返回的都是false。因为str.intern()返回的是"紫婆婆"这个常量值代表的字符串引用而非str。

为保证创建的字符串未曾在字符串常量池中出现过，可做如下修改：

```
public class Test {

    public static void main(String[] args) {
        String str = new StringBuilder("紫").append("婆婆").toString();
        System.out.println(str.intern() == str);
    }
}
```

在调用str.intern()时，"紫婆婆"并未被加入字符串常量池(此前出现的字符串常量只有"紫"及"婆婆")，因此JDK1.6会以"紫婆婆"为基础在方法区的字符串常量池中创建一个新的对象并返回该对象的引用，因其并非str，因此依然输出false。而对于JDK1.7，由于直接将str的引用加入到了字符串常量池并返回，因此输出true。

特别的，若代码如下：

```
public class Test {

    public static void main(String[] args) {
        String str = new StringBuilder("ja").append("va").toString();
        System.out.println(str.intern() == str);
    }
}
```

此时对于str的值"java"看似是在调用str.intern()时第一次出现在字符串常量池，但实际上此前像"java"这类有特殊含义的单词已然作为常量在字符串常量池中出现过了，只不过出现的地方并非用户程序所写的代码而已。因此此种情况实际相当于此前"紫婆婆"的例子，JDK1.6及JDK1.7均会输出false。

而到了JDK1.8永久代完全被元空间所替代。元空间是一块本地内存(Native Memory)，因此元空间的扩展极限是本机内存空间的扩展极限，垃圾收集器依然可以对其进行管理。自然，以前那些和永久代相关的设置参数及异常也不复存在了。

特别的，JDK1.8中字符串常量池依然在堆中。上文中关于String.intern()的测试代码在JDK1.8中的返回值均同JDK1.7。

方法区中最重要的一部分空间为运行时常量池(Runtime Constant Pool)。很多人将其与.class文件中的常量池(Constant Pool Table)等同看待，这是错误的：运行时常量池并不一定会读取常量池中的所有内容，它可能会根据本次加载的需求有所删减；同时除了编译期生成的.class文件外，运行期程序员也可以添加新的内容至运行时常量池(例如上文提到的JDK1.6及此前版本中的String类的intern方法)。

当方法区无法满足内存分配请求时会抛出OutOfMemoryError。拥有大量JSP或动态产生JSP文件的应用(JSP第一次运行时需要编译为Java类)容易触发这个Error。下面使用CGLib直接操作字节码在运行时生成大量的动态类来触发这个Error：

```
import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class Test {

    /**
     * -XX:PermSize=10m -XX:MaxPermSize=10m
     */
    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {
                @Override
                public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                    return proxy.invokeSuper(obj, args);
                }
            });
            enhancer.create();
        }
    }
}

class OOMObject {}
```

输出：

```
Exception in thread "main" 
Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "main"
```

# 本地内存

本地内存(Native Memory)并不是JVM运行时数据区域的一部分，JVM要操作这部分内存需要一些本地函数库的辅助。但是这块内存和JVM的内存管理又息息相关：例如上文提到的方法区中的元空间。

同样的例子还有JDK1.4中引入的NIO(New Input/Output)模式，该模式引入了一种基于通道(Channel)与缓冲区(Buffer)的I/O方式，它可以通过本地函数库直接分配本地内存，然后通过一个存于堆中的对象作为引用操作这块本地内存，这样就可以避免在Java堆及本地内存之间来回的复制数据，从而提升了程序的运行效率。

既然是内存空间，那么一定有其上限，对本地内存的申请超出其承载上限时会抛出OutOfMemoryError。

下面的例子越过了DirectByteBuffer类，直接通过反射获取Unsafe实例进行内存分配(Unsafe的getUnsafe()限制了只有引导类加载器才会返回实例，即Unsafe类设计者希望只有rt.jar中的类才能使用Unsafe的功能)。之所以这么做，是因为DirectByteBuffer类在分配内存时虽然也会抛出内存溢出异常，但是它并没有真正的向操作系统申请分配内存，而是通过计算得知内存不够了。作为操作系统而言并不知道曾有这样的一次内存分配操作。真正直接向操作系统申请分配内存的方法是Unsafe类的实例方法allocateMemory()。

```
import java.lang.reflect.Field;

import sun.misc.Unsafe;

public class Test {

    private static final int _1MB = 1024 * 1024;

    /**
     * -Xmx20m -XX:MaxDirectMemorySize=10m
     * 本地内存可由MaxDirectMemorySize指定，若未显式指定则与堆的最大值一致。
     */
    public static void main(String[] args) throws IllegalArgumentException, IllegalAccessException {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe)unsafeField.get(null);
        while (true) unsafe.allocateMemory(Test._1MB);
    }
}
```

运行结果

```
Exception in thread "main" java.lang.OutOfMemoryError
	at sun.misc.Unsafe.allocateMemory(Native Method)
	at com.test.Test.main(Test.java:19)
```