---
title: JVM-动态类型语言支持
date: 2017-12-08 11:08:49
tags: [Java,JVM]
categories: JVM
---

# 一些废话

对于是否要写这篇文章，我其实犹豫了很久。因为我个人对Java的这套动态类型语言当前的实现方案可以说是深恶痛绝，根本是连看都不想看到。此前我就曾在另一篇博文[JVM-类加载机制](/2017/11/23/JVM-类加载机制/)中狠狠的抱怨过：

*在此我想稍稍抱怨一下(当然，我不行，我也不上，我实在就是想bb两句)，自从我开始研究JVM起，invokedynamic指令连同它所希望实现的动态语言支持简直就像是个毒瘤，JVM规范在面对这个问题时总会变得扭曲不自然。逻辑变得复杂难懂倒还在其次，最重要的还是失去了缜密的美感，让人看着难受。细思其缘由，我想还是因为该功能实在是与Java设计的初衷背离太远所致：需求分析的时候完全没在这地方留灵活度，上线后发现需求迫切没办法强行加功能。Java语言自最初起就被设计为了静态类型语言，但是随着时代的发展，人们对动态类型的需求逐渐迫切，Java也总是会因此被人指为不灵活，逐渐要被时代所抛弃。Java也在竭力进行改良，其结果就是现在我们看到的动态语言支持方案。个人认为这个方案比较糟(我没有更好的方案，但就是感觉比较糟)，其与Java基本的结构完全不同，仅仅只是一个为了实现功能的异类。好了=-=，抱怨结束。*

<!-- more -->

不过抱怨归抱怨，该学的还是要学。难道JVM设计团队不清楚强行添加不合乎设计初衷的功能的后果吗？因为这实在是不得已而为之。当年高司令在为Java搭建基础架构时也不可能想到未来动态类型的需求会这么大。所以犹豫再三，我还是决定写下这篇博文，毕竟人生不如意事十之八九，很多事即便不想去做，也不得不做。

# 静态类型语言与动态类型语言

在介绍Java对动态类型语言的支持之前，自然要先介绍什么是动态类型语言。而为了能使得介绍更为全面，自然也要将相关的概念一并介绍才行。

首先是第一组概念：

动态类型语言(Dynamically Typed Language)：也叫动态语言。动态类型语言是指在运行期才去做数据类型检查的语言。声明变量时无需指明变量的数据类型，该语言会在第一次为变量赋值时，自动在内部将数据类型记录下来。Python和Ruby就是典型的动态类型语言。
- 静态类型语言(Statically Typed Language)：也叫静态语言。静态类型语言与动态类型语言刚好相反，编译期就会进行数据类型检查。换句话说，变量在声明时就需要指明数据类型。C/C++和Java均是典型的静态类型语言。

然后是第二组概念：

- 强类型语言(Strongly Typed)：一旦给一个变量赋上初值后，那么这个变量的数据类型就确定为这个初值的数据类型。除非发生强制类型转换，该变量的数据类型都不会发生变化，因此往往也会被称为类型安全的语言。Java就是典型的强类型语言。

- 弱类型语言(Weakly Typed)：数据类型可以被忽略的语言。它与强类型的语言相反，一个变量在其生命周期中可以被赋不同数据类型的值。因此往往也会被称为类型不安全的语言。

动态-静态与强类型-弱类型是两组容易被混淆的概念。事实上，二者的划分依据是不同的。动态-静态看的是变量初始时是否就需要确定数据类型，而强类型-弱类型看的是一旦变量被赋了初值后是否能发生变化。因为二者依据的是不同划分标准，所以共可能有4种排列组合结果：

![0.jpg](/images/blog_pic/JVM/动态类型语言支持/0.jpg)

上图中关于C++/C需要特别说明一下，这两个语言从语法层面看起来是强类型语言，然而其底层遵循的却是弱类型。因此业界对于这个问题其实比较模糊，没有什么明确的结论(注意不要拿这种无聊的问题和他人撕逼，C/C++更关心的是数据在内存中的长度，也就是更关心更底层，更本质的东西，其实并不特别在意数据类型，上图仅用于举例)。

以上是这些概念的抽象化描述。我们不妨再举一些实际的例子来说明各语言间因此可能产生的差异。

先来看下面这段Java代码：

```
int[] a = new int[-1];
```

这行代码是可以通过编译的，但是会在运行时抛出NegativeArraySizeException。JVM规范中明确规定NegativeArraySizeException是一个运行时异常，也就是说，所谓运行时异常，指得就是会抛出这种异常的代码不仅能通过编译，而且只要没被真正运行到，就不会抛出异常。与之相对的异常是连接时异常(例如NoClassDefFoundError)，连接时异常依然可以逃过编译，但是却无论如何也逃不过类加载的连接-解析阶段。通俗的说，就是指即便会抛出这种异常的代码存在于一条无法被执行到的分支路径上，照样也会抛出异常。其实这也很好理解：所谓连接时异常，指的自然就是在方法进行连接，也就是从符号引用到实际引用的过程中产生了异常，发生于类加载阶段，而对于每个需被加载到内存中的类而言，它根本不在乎代码是否在当前可能的执行路径上(事实上即便在乎也拿不到，除非程序实际运行到那行代码，否则类加载阶段是无法模拟预测程序会如何运行的)。因此这种异常一定会在类加载阶段被发现并抛出。

不过C语言中，含义相同的代码却无法通过编译：

```
int i[-1];
```

此时GCC会拒绝编译，报"size of array is negative"。

之所以会产生这种差异，主要是因为Java的编译期编译出的那个class文件并不是通常意义上的编译文件。C中编译期输出的结果直接就是最终结果了，能被实际执行指令的本地机器读取，自然要对结果负责。而Java编译出的这个class文件却只是供JVM使用的Java体系内部生成的字节码文件，要想让本地机器认识，还得在运行期翻译为本地机器的机器码。因此自然可以将部分所谓的这个编译期的工作挪到后续的运行期。

由上面的小例子我们可以得出以下推论：不同语言对数据的检查操作会发生在什么时期是没有固定的标准的，完全看语言本身的特性。所谓的"类型检查"也是同理。我们再看下面这段Java代码：

```
public class Test { 

    public static void main(String[] args) {
        obj.println("hello");
    }
}
```

如果全部代码只有这一行，那么显然是不行的，因为你没头没脑的冒出一个obj，JVM都不知道这个obj到底是什么，自然无法通过编译，会报obj cannot be resolved。为此，我们稍加修改：

```
import java.io.PrintStream;

public class Test { 

    public static void main(String[] args) {
        PrintStream obj = System.out;
        obj.println("hello");
    }
}
```

该代码会顺利运行并输出字符串hello。然后我们再修改一下代码：

```
import java.io.PrintStream;

public class Test { 

    public static void main(String[] args) {
        PrintStream obj = new T();
        obj.println("hello");
    }
}

class T {

    public void println() {}
}
```

此时将无法通过编译，并报Type mismatch: cannot convert from T to PrintStream。原因很简单：Java是静态类型语言，既然声明obj时规定了其静态类型为PrintStream，那么在赋初值时其实际类型就必须是PrintStream或PrintStream的孩子。其他类，即便其内部同样包含调用所需的println()也不行。

但是相同含义的代码在ECMAScript(即JavaScript)中却能正确运行。因为JavaScript是弱类型语言(同时也是动态类型语言)，其根本就没有变量声明时的静态类型这一说，在其生命周期中所指向的实际类型自然也可以随意的变化。当执行到调用println()这一行的代码时，只要obj此时所指向的实际类型中有符合条件的println()即可。换句话说，对于动态类型语言而言，变量本身是没有类型的，变量指向的值才有类型。

静态类型语言在编译期确定类型，最显著的好处是编译器可以提供严谨的类型检查，这样与类型相关的问题能在编译期就及时发现，利于稳定性及代码达到更大的规模。而动态类型语言在运行期确定类型，则可以为开发人员提供更大的灵活性。同时也能为代码"瘦身"。例如用Java写出的上百行代码用Python实现可能只需几十行，提升简洁性的同时可读性往往也更高，提高开发人员的开发效率。

# JDK1.7引入动态类型语言支持前的技术背景

自1996年JDK1.1诞生起，十余年间，其JVM的字节码指令集始终未发生任何变化。直至2011年JDK1.7发布，字节码指令集终于又迎来了一位新成员：invokedynamic指令。其目的就是为了使得JDK1.7支持本文所介绍的动态类型语言。也为JDK1.8可以顺利实现Lambda表达式打下基础。

本小节的标题是有讲究的。JDK1.7并未引入动态类型语言，而只是引入动态类型语言支持。换句话说，从本质上讲，Java依然是静态类型语言，无论动态类型的需求多么强烈，都不可能动摇Java的根本。Java只是在保证基本面不变的前提下，适当加入了支持动态类型的指令和逻辑。

本文开头说动态类型语言与Java的设计初衷相悖其实是有些绝对了。因为自打一开始Java体系就在构建两个无关性：平台无关性及语言无关性。Java体系的规范也被一分为二：Java语言规范与JVM规范始终都是独立实现的。其目的就是为了让JVM与Java解耦，JVM是运行Java的平台，却不仅仅是运行Java的平台，理论上只要符合它的class文件格式规范，它能容纳所有的编程语言，而这个"所有"当然是包括动态类型语言的。

所以说，并不能说最开始规划的时候完全没想到，而是规划的蓝图太过虚无缥缈(在JVM上支持一切语言，Java这是想千秋万载一统江湖吗)，导致执行的时候完全没遵循(其实还是然并卵)。在JDK1.7之前，无论是Java语言还是JVM走得都是坚定的静态类型道路。其对动态类型支持的缺失主要是体现在方法调用上。JDK1.7之前共有4条方法调用指令：

- 0xb6(invokevirtual):调用实例方法。会根据对象的实际类型进行动态单分派(虚方法分派)
- 0xb7(invokespecial):以操作数栈栈顶reference类型的数据所指向的对象为方法的接收者，调用此对象的实例构造器<init>方法，私有方法或超类构造方法。该指令的操作码之后会紧跟一个u2的操作数说明具体调用的是哪个方法，该参数指向常量池集合中的一个CONSTANT_UTF8_info类型的索引项，也就是该方法的方法符号引用
- 0xb8(invokestatic):调用类方法(static修饰的方法)
- 0xb9(invokeinterface):调用接口方法。运行期解释器会搜索一个实现了该接口方法的对象，并调用对应实现的接口方法

这4条指令接收的第一个参数都是被调用方法的符号引用(也就是类文件常量池中的CONSTANT_Methodref_info或CONSTANT_InterfaceMethodref_info类型的常量)。既然这个信息是在编译期生成的，那么其中记述的自然只能是调用该方法的对象的静态类型。而动态类型语言需要的却是对象第一次被赋值时的实际类型，并没有所谓的静态类型的概念。这样编译期就无能为力了，只能交由JVM在运行期来做。

虽然决定了JVM来做，但还有JVM怎么做的问题。一种思路就是让底层的改动尽量小一些。换句话说，最终调用方法的还是上文的那4条指令，JVM底层其实依然无法支持动态类型，只是在上层玩些骚操作，让使用者从结果上"看起来"是实现动态类型了。比如编译期编译时在class文件中留个占位符类型，运行期动态生成字节码得到实际类型并存入该占位符，然后在调用这些方法时，就不使用通用位置的符号引用了，而是使用这个占位符。

关于这种用伪物替换真物的行为，JDK1.5引入泛型时曾做过一次。其效果很不理想，因为假的就是假的，为了让底层架构的改动较小而使用了伪物，其结果就是为了让它能实现真物的功能而付出了巨大的代价。更糟糕的是，即便付出了代价，这个伪物所实现的功能依然是似是而非，直到现在Java的泛型还存在很多莫名其妙的规范，这些与其说是规范，更接近于bug，只是Java设计团队实在没法解了，便当作规范告诉开发人员不要这么做。

而在动态类型支持这件事上，JVM设计团队还是很明智的：使用真物。因为本来此前的设计思路就与动态类型不合，开发难度已然较大，要是再加上真物伪物的复杂性，这功能基本也就没法看了。

JDK1.7中，这个真物体现在JVM层面就是新增的invokedynamic指令，该指令为动态类型而生，直接支持动态类型。而在Java语言层面的体现则是新增的java.lang.invoke包。

# java.lang.invoke包

我们先来讲java.lang.invoke包，因为毕竟较之JVM，程序员还是对Java语言更有亲切感。

java.lang.invoke包(该包曾经历过几次改名，最开始很长一段时间内都叫做java.dyn，后来也曾短暂的改名为java.lang.mh，最后确定名称为java.lang.invoke)的主要目的是在之前单纯依靠符号引用来确定需调用的方法的方式之外，提供一种新的动态确定目标方法的机制，称为MethodHandle。

这与C/C++中的Function Pointer(函数指针)或C#中的Delegate类似。以C/C++中的函数指针为例，如果我们要实现一个带谓词的排序函数，常用做法是把谓词定义为函数，用函数指针把该谓词传递到排序方法，如下所示：

```
void sort(int list[], const int size, int (*compare)(int, int))
```

换句话说，这是把方法当成一个变量丢到另一个方法的入参里了。Java就做不到这一点。如要在Java中实现类似的功能，则需要绕一些远路：通常是设计一个带有compare()方法的Comparator接口，然后将实现了这个接口的对象传入排序函数，排序函数内部再回调该对象的compare()方法。我们平时最常用的java.util.Collections类的:

```
public static <T> void sort(List<T> list, Comparator<? super T> c)
```

其内部就是按照这个思路实现的。

不过，在JDK1.7引入MethodHandle后，Java也拥有类似于C++的函数指针的功能了。

```
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;

public class Test { 

    private static final MethodHandle getPrintlnMH(Object receiver) throws NoSuchMethodException, IllegalAccessException {
        /*
         * MethodType代表方法类型
         * 其类方法methodType()：
         * 第一个参数：方法的返回值
         * 后续参数：方法接收的参数列表
         */
        MethodType mt = MethodType.methodType(void.class, String.class);
        /**
         * MethodHandles.lookup().findxxx()方法的作用为在指定类中找到符合方法签名，返回值及访问权限要求的方法句柄
         * 具体到本例中，因需要调用的是虚方法。故调用findVirtual()方法。其参数含义为：
         * 参数1：方法接收者所属类。遵正常渠道调用的虚方法编译器会隐式添加指向接收者的this指针，此处则需要我们自己指定。
         * 参数2：需调用虚方法的简单名称
         * 参数3：需调用虚方法的返回值及参数列表
         * 最终，还需将该方法再由bindTo()方法显式绑定回接受者上
         */
        return MethodHandles.lookup().findVirtual(receiver.getClass(), "println", mt).bindTo(receiver);
    }

    public static void main(String[] args) throws Throwable {
        String str = "我来自幻想乡";
        Test.getPrintlnMH(System.out).invokeExact(str);
        Test.getPrintlnMH(new Marisa()).invokeExact(str);
    }
}

class Marisa {

    final void println(String str) {
        System.out.println(str + "DAZE");
    }
}
```

输出：

```
我来自幻想乡
我来自幻想乡DAZE
```

本质上来讲，本例中的getPrintlnMH()模拟了invokevirtual指令的执行过程，只不过它的分派逻辑并非固化在Class文件的字节码上，而是通过getPrintlnMH()这个具体的方法由程序员指定实现。getPrintlnMH()的返回值(MethodHandle)即可视为对最终调用方法的一个引用。

再回到上文中比较的那个例子，有了MethodHandle后，我们就可以使用类似于如下方法在Java中实现比较：

```
void sort(List list, MethodHandle compare)
```

本质上来说，Java依然无法接收方法为参数，它所采用的方式为将方法的必要信息包装成了一个对象，然后像使用方法那样使用这个对象。

以上就是MethodHandle的基本用途：封装一个代表这个方法的对象，然后在实际调用时以该对象所代表的方法信息为依据调用具体方法。这样我们不禁会产生一个新的疑问：所实现的功能不就是反射吗？为什么不直接用反射机制完成呢？为什么要再搞出个MethodHandle呢？

确实，仅从Java语言规范的角度上讲，MethodHandle所实现的功能与反射有很大重叠的部分，然而它们还是有以下区别：

首先，反射和MethodHandle虽然都是在模拟方法调用，然而二者所模拟的层次不同：反射是在模拟Java代码层次的方法调用，不会去关心底层字节码的实现机制。而MethodHandle是在模拟字节码层次的方法调用。MethodHandles.lookup()中共有3个方法：

- findStatic():对应invokestatic指令
- findSpecial():对应invokespecial指令
- findVirtual():对应invokevirtual及invokeinterface指令

其次，反射中负责方法调用的是java.lang.reflect.Method，MethodHandle中负责方法调用的是java.lang.invoke.MethodHandle。前者包含了这个方法所有的信息，而后者仅包含与方法调用相关的信息。可以这么认为：Method是重量级，MethodHandle是轻量级。

最后，由于MethodHandle是对字节码指令的直接模拟，因此JVM对字节码指令做的种种优化(例如方法内联)理论上都适用于MethodHandle。虽然现在尚不完善，但仍留下了可供优化的空间。而仅限于语言层面的反射则做不到这一点。

通过上文的这3点，我们可以总结出：MethodHandle与反射最本质的区别就是二者作用的层级不同。反射作用于Java语法层面，因此只能为Java语言服务。而MethodHandle则作用于JVM的字节码指令层面，可以为包含Java在内的一切运行于JVM之上的语言服务。

# invokedynamic

在介绍完Java语言层面对动态类型的支持之后，终于来到了JVM层面。

从本质上来讲，invokedynamic指令与MethodHandle机制的目的是一样的：都是为了解决原有的4条invokexxx指令方法分派规则固化在JVM之中的问题。从而将如何定位目标方法的决定权从JVM转移到具体的用户代码中，让开发人员(也包括运行于JVM上的语言的设计人员)有更高的灵活度。

因此，我们可以将invokedynamic指令与MethodHandle机制视为为实现同一个目标的两种具体的做法。MethodHandle机制是采用上层Java代码及API实现的，而invokedynamic指令则是直接通过字节码指令及class文件实现的。二者在设计思路上有很多共通之处。

每一处含有invokedynamic指令的位置都被称作动态调用点(Dynamic Call Site)。invokedynamic指令的第一个参数不再是代表方法符号引用的CONSTANT_Methodref_info常量，而是变为JDK1.7新加入的CONSTANT_InvokeDynamic_info常量。从这个常量中可以得到如下3项信息：

- 引导方法(Bootstrap Method):实际存储于同样新增的class文件-属性表集合-BootstrapMethods中。其有固定的入参，返回值为java.lang.invoke.CallSite对象，该对象代表真正要执行的目标方法(类似于MethodHandle机制的MethodHandle对象)。
- 方法类型(MethodType)
- 名称

根据CONSTANT_InvokeDynamic_info常量中提供的信息，JVM就可以找到并执行引导方法(这样看来，引导方法的作用类似于上文中我们实现MethodHandle机制时设计的类方法getPrintlnMH())，最终利用引导方法返回的CallSite对象调用目标方法。

因为Java语言依然是静态类型的语言，因此与MethodHandle机制所不同的是，invokedynamic指令所面向的使用者并非Java语言，而是那些运行于JVM之上的动态类型语言。或者更具体的说，javac编译器是无法生成带有invokedynamic指令的class文件的。在Java语法层面曾经有一个java.dyn.InvokeDynamic的语法糖可以实现，但是后来取消了。

或者我们可以把话说的更明白一些，如果仅仅是对于学习Java这一门语言而言，invokedynamic指令是没有用处的，可以当它不存在。这也是JVM关于语言无关性的设计初衷：Java语言是运行于JVM平台上的，然而JVM并非完全就是为了支持Java语言而存在的。