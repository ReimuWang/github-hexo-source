---
title: JVM-类加载机制
date: 2017-11-23 10:30:49
tags: [Java,JVM]
categories: JVM
---

JVM将描述类的数据从class文件加载到内存，并对数据进行一系列处理，最终形成可被JVM直接使用的Java类型，这就是JVM的类加载机制。

与那些在编译期就进行连接操作的语言不同，Java中的类型的加载，连接和初始化都是在运行期完成的(实际上这些语言的编译和Java的编译指的也不是同一回事，这些语言编译后直接就生成硬件机器所需的机器码了，自然要完成连接。而Java编译后得到的仅仅是JVM才看得懂的字节码，只是一个半成品)。这样做会增大运行期的性能开销，相应的也为Java提供了高度的灵活性：Java天生就具有的可以动态扩展的语言特性依靠的就是运行期动态加载及动态连接。例如，假设我们编写了一个面向接口的Java应用，那么我们可等到运行期再指定其具体的实现类。再比如，我们可以通过Java预定义的或自定义的类加载器，让一个本地Java应用可以在运行期从网络或其他地方动态加载一段二进制字节码流作为其程序的一部分。

为行文方便，在此为后文的叙述做如下规定：

- 每个具体的class文件既可能代表一个类，也可能代表一个接口。后文将统一以类来代指类和接口这两种情况。而对于类和接口需区分对待的场合会特别声明。
- 所谓的class文件只是一种约定俗成的称呼，实际上，class文件并不一定非要是存储于磁盘中的文件，只要是一段符合class规范的二进制字节流即可，无论以何种形式存在均可。

<!-- more -->

# 类加载时机

在介绍类加载过程之前，首先需要明确类加载的时机，即什么情况才会触发JVM去加载一个类？JVM规范对此并没有做出强制约束，即实际的加载时机全凭具体的JVM实现自行发挥。

# 类加载过程概览

类从被加载到JVM内存开始，到卸载出内存为止，整个生命周期会经历如下5个大的阶段：

1. 加载(Loading)
2. 连接(Linking)
3. 初始化(Initialization)
4. 使用(Using)
5. 卸载(Unloading)

而步骤2连接又可细分为如下3步：

1. 验证(Verification)
2. 准备(Preparation)
3. 解析(Resolution)

拆分后具体可如下图所示：

![0.png](/images/blog_pic/JVM/类加载机制/0.png)

在分解而得的这7个小步骤中，加载，验证，准备，初始化，使用，卸载这6个步骤的开始顺序是严格有序的，而解析则不一定：某些情况下其可能会在初始化开始之后再开始，这是为了支持Java语言的运行时绑定(也称为动态绑定或晚期绑定)。注意，本段文字在描述顺序时使用的均是"开始"，而非"进行"或"完成"。即这些步骤即便严格有序也是开始的严格有序，往往是各步骤交叉混合运行的，即不会等待一个步骤彻底执行完成再执行下一个步骤，而是通常会在一个步骤的执行过程中调用，激活下一个步骤。

上文提到过，JVM规范并未对类加载时机做强制约束，但是对类加载步骤中的初始化却做了约束，这其实也算是变相约束了类加载时机，因为想要开始初始化，则必须先开始其前置的加载，验证，准备(如前文所述，解析不一定)。这实际上就是在要求类必须要被加载了。具体的，必须开始初始化的时机有以下5种：

**时机1**

执行以下4条字节码指令时，若对应的类没有初始化，则需要触发其初始化：

1. **new**：例如使用new关键字实例化对象
2. **getstatic,putstatic**：例如读取或设置一个类的类变量(被final修饰且数据类型为基本类型或java.lang.String的类变量除外。因为其值已在编译期存入class文件-->字段表集合-->字段表-->属性集合-->ConstantValue属性中，会通过常量传播优化存入调用代码中)
3. **invokestatic**：例如调用一个类的类方法

这里需要说明一下的是第2点。我们不妨来看一段代码。

首先给出O：

```
public class O {

    public static String V = "Reimu";

    static {
        System.out.println("O init");
    }
}
```

随后是测试类：

```
public class Test {

    public static void main(String[] args) {
        System.out.println(O.V);
    }
}
```

输出为：

```
O init
Reimu
```

O被初始化了。我们不妨将Test用javap反编译：

```
Classfile /E:/Test.class
  Last modified 2017-11-23; size 439 bytes
  MD5 checksum 2fcd45f3efed2349f7934e5b3857ce92
  Compiled from "Test.java"
public class Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#15         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = Fieldref           #18.#19        //  O.V:Ljava/lang/String;
   #4 = Methodref          #20.#21        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #22            //  Test
   #6 = Class              #23            //  java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Test.java
  #15 = NameAndType        #7:#8          //  "<init>":()V
  #16 = Class              #24            //  java/lang/System
  #17 = NameAndType        #25:#26        //  out:Ljava/io/PrintStream;
  #18 = Class              #27            //  O
  #19 = NameAndType        #28:#29        //  V:Ljava/lang/String;
  #20 = Class              #30            //  java/io/PrintStream
  #21 = NameAndType        #31:#32        //  println:(Ljava/lang/String;)V
  #22 = Utf8               Test
  #23 = Utf8               java/lang/Object
  #24 = Utf8               java/lang/System
  #25 = Utf8               out
  #26 = Utf8               Ljava/io/PrintStream;
  #27 = Utf8               O
  #28 = Utf8               V
  #29 = Utf8               Ljava/lang/String;
  #30 = Utf8               java/io/PrintStream
  #31 = Utf8               println
  #32 = Utf8               (Ljava/lang/String;)V
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

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: getstatic     #3                  // Field O.V:Ljava/lang/String;
         6: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         9: return        
      LineNumberTable:
        line 4: 0
        line 5: 9
}
```

很显然，main()方法调用的仍是类O的类变量V，因此此时类必须初始化。


然后我们对O做如下修改，即给V加上final修饰符：

```
public class O {

    public static final String V = "Reimu";

    static {
        System.out.println("O init");
    }
}
```

测试类不变，输出变为：

```
Reimu
```

此时O就没有初始化。这其实是Javac编译阶段的常量传播优化：O中被final修饰的类变量V已在编译期被存储到Test类的常量池中。此后Test中对O.V的访问实际都是在对自身常量池的访问，自然也就不需要再初始化O了。我们不妨再用javap对此时的Test.class进行反编译：

```
Classfile /E:/Test.class
  Last modified 2017-11-23; size 407 bytes
  MD5 checksum fa5eb1af23ee221917b59def8993a5b5
  Compiled from "Test.java"
public class Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#15         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            //  Reimu
   #4 = Methodref          #19.#20        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            //  Test
   #6 = Class              #22            //  java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Test.java
  #15 = NameAndType        #7:#8          //  "<init>":()V
  #16 = Class              #23            //  java/lang/System
  #17 = NameAndType        #24:#25        //  out:Ljava/io/PrintStream;
  #18 = Utf8               Reimu
  #19 = Class              #26            //  java/io/PrintStream
  #20 = NameAndType        #27:#28        //  println:(Ljava/lang/String;)V
  #21 = Utf8               Test
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
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

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Reimu
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return        
      LineNumberTable:
        line 4: 0
        line 5: 8
}
```

注意常量池中的18，此时Reimu已被加入了Test的常量池中，main()方法中对O.V的调用实际上也是指向Test自身常量池中的常量，已与类O没有关系了。

**时机2**

使用java.lang.reflect包的方法对类进行反射调用时，若类没有进行过初始化，则需要先触发其初始化。

**时机3**

初始化一个类时，若发现其父类尚未进行过初始化，则需要先触发其父类的初始化。

**时机4**

当程序启动时，用户需要指定一个作为入口的主类(即包含main方法的那个类)，JVM会先初始化这个主类。

**时机5**

当使用JDK1.7的动态语言支持时(即invokedynamic指令和BootstrapMethods，截至JDK1.7为止，javac编译器尚无法生成invokedynamic指令及BootstrapMethods属性，必须通过一些非常规的手段才能使用到它们)，如果一个java.lang.invoke.MethodHandle实例最后的解析结果是REF_getStatic,REF_putStatic,REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

**主动引用及被动引用**

对于以上5种初始化时机，JVM规范给出了一个很强烈的限定语：有且只有。这5种场景中的行为称为对一个类进行主动引用。除此之外，所有引用类的方式都不会触发类的初始化，称为被动引用。

关于主动引用及被动引用很容易被误判，以如下代码为例：

```
public class Test {

    public static void main(String[] args) {
        System.out.println(Son.V);
    }
}

class Super {

    public static int V = 123;

    static {
        System.out.println("Super init");
    }
}

class Son extends Super {

    static {
        System.out.println("Son init");
    }
}
```

运行后输出如下：

```
Super init
123
```

结果只输出了Super init而没有Son init。说明只有父类得到了初始化，子类却没有。这印证了前文的规范：子类调用父类中的静态变量并不在上文"有且只有"的那5种情况中。至于子类有没有经历加载和验证，准备等初始化的前置阶段，JVM规范则未明确规定，可供具体实现自行发挥。

再看如下代码，我们先定义类O：

```
package com.test;

public class O {

    static {
        System.out.println("O init");
    }
}
```

随后再来一个测试类：

```
package com.test;

public class Test {

    public static void main(String[] args) {
        O[] a = new O[5];
    }
}
```

执行后，程序没有任何输出。说明com.test.O并没有初始化。然而，在我们看不到的地方，实际上触发了另一个名为[Lcom.test.O的类的初始化。很显然，这并不是一个合法的类的全限定名，而是类似于数组变量的描述符。它是一个由JVM自动生成，直接继承于java.lang.Object的子类，创建动作由字节码指令newarray触发。[Lcom.test.O代表了一个元素类型为com.test.O的一维数组。数组对象应有的属性及方法均实现在该类中，对于用户而言，这个类可直接使用的只有被修饰为public的length字段及clone()方法。Java对数组的这种严密的封装方式也是Java在访问数组时较之C或C++相对安全的原因：使用C或C++时，程序员对数组的操作直接被翻译为了对数组指针的移动，万一越界了那就真的是在底层越界了。而Java对数组的操作却隔着一个JVM自造出来的数组对象，接触不到底层实际用于存储数据的数组。即便越界也是Java语言层面的越界，顶多也就是抛出越界异常java.lang.ArrayIndexOutOfBoundsException，一切问题都会限制在JVM内部，而不会真的影响到更底层的内存(更准确的说，数组越界检查并非封装在数组类中，而是封装在该数组类进行数组操作时访问的xaload,xastore指令中)。

接口的加载过程则与类稍有不同。同类一样，接口的加载也分为上文介绍的那些步骤。不过前文我们在判断一个类是否初始化时，使用了static{}代码块，若该代码块被调用则类被初始化，反之则没有。接口中是不能声明static{}代码块的，但是在底层，JVM其实依然会为接口生成类构造器方法&lt;clinit&gt;用于初始化接口所定义的静态变量。初始化的时机依然是前文介绍的那"有且只有"的5种。与类有细微差别的是第3点：

```
3. 初始化一个类时，若发现其父类尚未进行过初始化，则需要先触发其父类的初始化。
```

对于接口而言，这条约束就没有那么严格了。即一个接口在初始化时，除非真的用到了父接口(例如引用父接口中定义的静态变量)，否则不需要初始化父接口。

接下来将逐个依序详解类加载的前5个步骤，也就是加载，验证，准备，解析和初始化。至于最后两项使用和卸载则没什么可说的。

# 加载

首先必须明确的是，加载(Loading)是类加载(Class Loading)的第一个阶段，二者之间是从属关系。不要有所混淆。

根据JVM规范，加载阶段JVM需要依序完成如下3件事：

1. 通过类的全限定名获取该类的二进制字节流。
2. 将这个字节流所代表的静态存储结构加载入JVM所管理的内存，转化为方法区中的运行时数据结构(这里与验证阶段有重叠，实际上完成转化是在验证的第一阶段文件格式验证完成后)。
3. 在Java堆上创建一个代表该类的java.lang.Class类的实例，作为方法区这个类的各种数据的访问入口。换句话说，类信息本身存储在方法区中，然后在Java堆上为该类专门建立了一个类对象，像句柄那样作为中间人管理及操作该类。

很显然，JVM规范对于加载的实现的要求并不具体，简直连产品经理都算不上，仅仅只是用户在描述需求。通篇都是"你要做到xxx"，至于怎么实现则几乎完全没有提及。因此对于JVM实现者而言，这种规范的灵活性就很高，例如，第一点"通过类的全限定名获取该类的二进制字节流"，一没有指明要从哪里获取，二没有指明具体该怎么获取。

JVM规范对于加载阶段的低限制给JVM实现者提供了广阔的自行发挥的余地。许多重要的Java技术都建立在这一基础之上，例如依然针对第一点"通过类的全限定名获取该类的二进制字节流"：

- 从zip包中直接读取二进制字节流。这也是日后Java特有的JAR,EAR,WAR压缩包的基础。
- 从网络中读取二进制字节流，最典型的应用场景就是Applet。
- 运行期通过计算动态生成，最典型的应用场景就是反射中的动态代理技术，即java.lang.reflect.Proxy.newProxyInstance()方法所动态生成的那个代理类的二进制字节流。
- 由其他类型的文件生成。最典型的应用场景是.jsp文件。运行期.jsp文件会生成对应Java类的二进制字节流。
- 从数据库中读取，这种读取方式的应用场景相对较少，主要用于某些大型分布式中间件服务器(例如SAP Netweaver)，其基本思路为将程序二进制字节码存储到数据库中并在集群中的各机器间进行分发。

相对于类加载机制中的其他阶段，非数组类的加载阶段(或者更具体的说，是加载阶段的第一步，通过类的全限定名获取该类的二进制字节流)是JVM规范限制最少的，也是开发人员可以玩出最多花样的。Java中大部分在运行期动态完成的骚操作依托的都是这一阶段。因为加载阶段既可以使用系统预设的引导类加载器完成，也可以使用用户自定义的类加载器(即重写类加载器的loadClass()方法)来控制字节流的获取方式。

而数组类则有所不同，如前文所述，数组类本身并不是由类加载器创建的，而是由JVM直接自动生成的。但数组类依然与类加载器有着千丝万缕的联系。因为即使数组类本身不是由类加载器创建，然而其中的元素却依然还是类加载器创建的(不考虑元素依然是数组的情况。其实即便元素是数组，即便元素的元素依然是数组，最终的那一层元素总得是非数组类，数组去掉所有维度信息后剩下的最终那一层元素被称为该数组的元素类型(Element Type))。

数组去掉一个维度后的元素类型被称为组件类型(Component Type)，数组的可见性与其组件类型相同。若组件类型为基本数据类型，则数组的可见性默认为public。

若数组的组件类型是基本数据类型，则该组件类型将使用Java预设的引导类加载器加载。该数组也会被标记为与引导类加载器关联。

若数组的组件类型是引用数据类型，则该组件类型将使用该引用类型的类加载器加载。该数组也会被标记为与该类加载器关联。

加载与验证的部分内容(如一部分字节码文件格式的验证动作)是交叉进行的，这些验证动作本质上依然属于验证阶段。可以理解为验证阶段认为时机已然成熟了，无需等待加载完成就可以开始了。

# 验证

如果说加载是把所需的类信息读入内存，那么紧随其后的连接就是实际开始干活了。而连接的第一步验证则类似于程序入口处的边界判断。这一阶段的目的是为了确保class文件的字节流中包含的信息符合当前JVM实现的读取规范，并且不会危害到JVM自身的安全。

Java语言其实是一门语法上相对安全的语言(依然是较之C与C++)，语法上对编写者的限制较多，使用纯粹的Java代码是无法做到诸如访问数组边界以外的数据，将一个对象强制转型为它无法实现的类型，跳转到不存在的代码行等。如果强行这么做了，javac编译器也会拒绝编译。因此如果所有class字节码都是由javac编译器编译源码而得的，那么其实验证阶段就不那么必要了。

然而JVM的无关性却被设计为既有平台无关性，也有语言无关性。因此和JVM相关的只有class文件字节流，至于该字节流是怎么来的，是由javac编译器生成的，还是由其他编译器生成的，甚至是直接编写二进制文件硬写出来的，它并不关心。因此在字节流层面，上述通过Java语法无法实现，过不了javac编译器这一关的错误操作在字节码层面都是可以实现的，至少在语义上可以表达出来。因此，JVM在实际干活前必须检查读入的字节流是否安全。

验证阶段是非常重要的，该阶段几乎直接决定了具体的JVM实现承受恶意代码攻击的健壮性。从执行性能的角度来看，验证阶段的工作量占了整个类加载全过程中很大的一部分。

规范通常是不干预安全性这种和实际功能无关的模块的，因此JVM规范对于验证阶段的指导及限制很是笼统。最为基本的就是对class文件格式的约束。这也是最基本的约束。如果验证到字节流不符合格式上的规范，JVM就会抛出一个java.lang.VerifyError异常或其子类异常。在此之上的，应检查哪些方面，如何检查，何时检查，检查出的问题什么程度的能忍，什么程度不能忍都没有具体指示。其只是从验证思路上，将本阶段大致分为以下4种检验动作：

1. 文件格式验证
2. 元数据验证
3. 字节码验证
4. 符号引用验证

**文件格式验证**

这也是最基本的验证内容，且不论内容是否正确安全，如果连格式都不符合规范，该JVM从结构上都无法处理该字节流，那么后续的也都不用验了。摘录部分这一阶段的验证点如下：

- 开头的魔数是否是0xCAFEBABE
- 主次版本号是否在当前JVM可处理的范围之内
- 检查常量tag标志，判断常量池集合中是否有不被支持的常量类型
- 指向常量池集合中的常量的索引是否指向了不存在的常量或错误类型的常量
- 被定义为CONSTANT_Utf8_info类型的常量中的值是否符合Utf8压缩编码的规范
- class是否缺少了必要的信息或被添加了莫名其妙不认识的信息

之后还有很长的验证点。不过至此我们已可以看出，该步确实就是在顺着class文件规范逐项严格检验是否符合要求。

本步完成前，字节流依然是以原始流的信息被临时读入内存，本步完成后，该字节流才真正的被映射为方法区中的结构。自本步后的后续步骤操作的都是方法区中对应的该类的信息了，和原始的字节流就没有关系了。这点并不难理解：只有格式符合规范，才能正确的映射。因此欲完成字节流向方法区中数据的转换，必须先完成文件格式验证。

**元数据验证**

第一阶段文件格式验证验证的是格式是否符合规范。而第二阶段元数据验证则更进一步，验证的是语义是否符合规范。即验证此时方法区中该类的信息是否符合Java的语言规范。截取部分验证点举例如下：

- 这个类是否有父类(除了java.lang.Object外，所有类都应有父类)。
- 这个类所在的继承链中是否有不允许被继承的类(即被final修饰的类)？
- 如果这个类不是抽象类，那么它是否实现了其父类或其所实现的接口中要求它实现的所有方法？
- 类中的字段，方法是否与其父类有冲突(例如覆盖了父类的被final修饰的实例变量或实例方法)。

**字节码验证**

元数据验证所做的语义验证针对的是一个个独立的个体，而本步字节码验证针对的则是方法体中的执行逻辑。这也是整个验证阶段最难于验证，最复杂的一个环节。其主要通过模拟代码执行过程中的数据流及控制流，确定整个过程是符合逻辑且安全的。截取部分验证点举例如下：

- 确保任何时刻操作数栈栈顶1个或多个元素的类型与当前正欲执行的字节码指令所需的操作数匹配。
- 确保跳转指令不会跳转出方法体或者跳转到一个奇怪的不能跳转的位置。
- 确保方法体中的类型转换是有效的。例如我们可以将一个子类对象赋值给其父类数据类型。但是将一个父类对象赋值给其子类类型甚至是完全不相关的类型则不行。

很显然，通过程序去验证程序的逻辑是否正确且安全是永远无法做到绝对准确的。同时繁杂的检查也将极大的占用运行时间。我们无法保证绝对准确，但是却可以尽可能的通过将验证操作前移至编译阶段来缩短运行期的验证时间(毕竟编译只需要一次，再慢也等得起)。JVM团队将方法按控制流段落划分为一个个基本块(Basic Block)，同时在class文件-->方法表集合-->方法表-->属性集合-->Code属性-->属性集合-->StackMapTable属性中存储每个基本块开始时局部变量表及操作数栈应有的状态。这样，在运行期类加载-->验证-->字节码验证阶段，就不需要根据数据流和控制流再次推算这些应有的状态了。转而直接比对实际模拟结果是否与StackMapTable属性中存储的状态一致即可。当然，使用StackMapTable属性在减少验证时间的同时，也增大了出错的几率：要是StackMapTable属性的计算出错了呢？纯粹因为计算失误导致出错的可能性不大(因为我们可以认为即便将这个验证前移到了编译阶段，验证的算法都是一样的。编译期要是算错了，放到运行期一样会算错)，问题在于恶意伪造StackMapTable属性从而骗过JVM。这个风险现在无解，是为了降低验证时间必须要承担的。

**符号引用验证**

符号引用验证发生在连接的第三个阶段：解析(很显然，又交叉执行了)。具体来说，是发生在JVM将符号引用转化为直接引用时。简单来说，进行到这一步，类的模版准备工作也快完事了。和C及C++相比，Java仅仅只是把连接操作后移到了运行期的解析阶段，并非就不进行连接了：因为不连接就无法指向实际的对象位置，光靠一个符号标记是无法运行下去的。而符号引用验证就是在验证该类常量池集合中自身以外的那些符号引用到底是不是真的存在，存在了到底允不允许本类用...截取部分验证点举例如下：

- 通过符号引用所描述的全限定名是否能找到对应的类，找到后是否有权限访问。  
- 类中是否有所需的字段或方法，这些字段或方法是否有访问权限。

显然，本步验证是在为类加载-连接-解析做准备，若无法通过符号引用验证，则会抛出一个java.lang.IncompatibleClassChangeError的子类异常，如java.lang.IllegalAccessError,java.lang.NoSuchFieldError,java.lang.NoSuchMethodError等。

**总结**

从安全的角度来看，验证阶段至关重要；但是仅从程序实现角度来看，验证阶段则完全没必要：因为其中没有任何用户逻辑，无法为用户创造具体价值。因此，若对class文件充分信任，则可以在运行时通过-Xverify:none配置关闭大部分的验证操作，加速程序运行。

# 准备

连接的第二个阶段名为准备。该阶段正式在方法区中为类变量分配内存并设置其初始值。有两点需要注意：

首先，分配内存的是类变量，实例变量会在对象实例诞生时随对象在Java堆上初始化。这个很好理解：类变量是类模板的一部分，自然应随模版初始化于方法区，而实例变量属于每个实例，自然会随实例初始化于Java堆上。

其次，这里设置的初始值通常指的是对应类型的零值，而非用户指定的具体值。如有以下类变量：

```
public static int VALUE = 123;
```

则在准备阶段该类变量会在方法区中初始化，其值为0而非123。可以理解为该阶段仅仅只是JVM为类开辟空间的准备阶段(一如其名)，既然开辟了空间那么当然要给个默认的初值才好。至于用户指定的值123则存放于类构造器方法&lt;clinit&gt;中，会在类加载-初始化阶段由putstatic指令赋值。Java中各类型的零值如下图所示：

![1.png](/images/blog_pic/JVM/类加载机制/1.png)

上文在描述初始值时使用了"通常"，那么自然就会有"不通常"，即类变量(基本数据类型或字符串)被final修饰：

```
public static final int VALUE = 123;
```

其值已在编译期存入class文件-->字段表集合-->字段表-->属性集合-->ConstantValue属性中，则准备阶段直接就会把该类变量初始化为ConstantValue中的值。这也很好理解：毕竟，被final修饰的变量无法再被修改了，那么即使是到了初始化阶段，或是之后的任何阶段也都玩不出什么别的花样了，始终就会保持一个值。因此也就没必要费两遍事，直接在准备阶段赋了最终值即可(这也是final关键字的精髓所在，一如其名，最初即为最终)。

# 解析

解析是连接的最后一个阶段，也是类加载过程中至关重要的一个阶段。解析会将常量池内的符号引用替换为直接引用。在[JVM-类文件结构](/2017/11/07/JVM-类文件结构/)中，符号引用可以是CONSTANT_Class_info,CONSTANT_Fieldref_info,CONSTANT_Methodref_info等类型的常量。现在再从符号引用和直接引用对比的角度回顾下符号引用，并引入直接引用：

- 符号引用(Symbolic References):符号引用就是用一组符号描述所引用的目标，理论上并没有什么特别硬性的规定。它可以是任何形式的字面量，只要指定好描述规范(该规范已在JVM规范中指定，因此对于具体JVM实现而言，其实还是有规范的)，能在解析时无歧义的定位到目标即可。符号引用是一个逻辑上的，抽象的概念，被引用的目标不一定已加载到内存中(实际上是否加载它根本不在乎)。其与JVM实现具体的内存布局无关，由于JVM规范已规定了符号引用的描述规范(不规定不行，因为不规定的话就无法让一个统一的class文件中的符号引用在各JVM实现上无障碍的运行)，因此各JVM解析符号引用的规范必须是一致的，遵守JVM规范的。
- 直接引用(Direct References):直接引用可以是直接指向目标的指针，相对偏移量，也可以是一个能间接定位到目标的句柄。直接引用所指向的必然是一个实际已然存在的目标，其和具体JVM实现的的内存布局有关，即同一个符号引用在不同的JVM实例上翻译出的直接引用一般不会相同(其实即便同一个实例两次运行一般也不会相同)。

JVM规范并未指明解析具体的触发时机，只是粗略的规定在执行以下这16个用于操作符号引用的字节码指令之前，必须要对它们使用的符号引用进行解析(即保证符号引用确实是可用的，并指到实际工作的目标上)：

- 0xbd(anewarray):创建一个指定引用类型(如类，接口，数组)的数组，并将其引用值压入操作数栈
- 0xc0(checkcast):检验类实例的类型转换，检验未通过将抛出ClassCastException
- 0xb4(getfield):获取指定类的指定实例的实例变量，并将其值压入操作数栈
- 0xb2(getstatic):获取指定类的指定类变量，并将其值压入操作数栈。该指令的操作码之后会紧跟一个u2的操作数说明具体需要的是哪个类变量，该参数指向常量池集合中的一个CONSTANT_UTF8_info类型的索引项，也就是该字段的字段符号引用
- 0xc1(instanceof):检验类实例是否是指定类的实例，如果是将1压入操作数栈，反之压入0
- invokedynamic:尚未出现在字节码指令集中
- 0xb9(invokeinterface):调用接口方法。运行期解释器会搜索一个实现了该接口方法的对象，并调用对应实现的接口方法
- 0xb7(invokespecial):以操作数栈栈顶reference类型的数据所指向的对象为方法的接收者，调用此对象的实例构造器&lt;init&gt;方法，私有方法或超类构造方法。该指令的操作码之后会紧跟一个u2的操作数说明具体调用的是哪个方法，该参数指向常量池集合中的一个CONSTANT_UTF8_info类型的索引项，也就是该方法的方法符号引用
- 0xb8(invokestatic):调用类方法(static修饰的方法)
- 0xb6(invokevirtual):调用实例方法。会根据对象的实际类型进行动态单分派(虚方法分派)
- 0x12(ldc):将int,float或String型常量值从常量池中推送至操作数栈栈顶。该指令的操作码之后会紧跟一个u2的操作数作为具体的值，该参数指向常量池集合中的一个对应类型的索引项
- 0x13(ldc_w):将int,float或String型常量值从常量池中推送至操作数栈栈顶(宽索引)
- 0xc5(multianewarray):创建指定类型和指定维度的多维数组(执行该指令时，操作栈中必须包含各维度的长度值)，并将其引用值压入操作数栈
- 0xbb(new):创建一个对象，并将其引用值压入操作数栈
- 0xb5(putfield):为指定类的指定实例变量赋值
- 0xb3(putstatic):为指定类的指定类变量赋值

JVM规范的这一限制只是限制了解析符号引用的最晚时机：即当这些指令执行时符号引用必须已经转为直接引用。换句话说，具体的JVM实现自然也可以根据需求在更早的时候完成这一操作。

对一个符号引用进行多次解析请求是很常见的事。除invokedynamic指令之外，JVM实现可选择将第一次解析的结果进行缓存(比如，在方法区的运行时常量池中记录直接引用，并把该常量标志为已解析状态)，这样除非发生特定的直接引用位置变动的情况，大部分情况都可以避免重复进行解析操作。无论如何，JVM规范都要求保证的是在同一个实体中，如果一个符号引用之前已被成功解析过，那么后续的引用解析请求就应当一直成功。同样的，若第一次解析失败了，那么其他指令对这个符号引用后续的解析请求也应收到相同的异常。

invokedynamic指令则不遵循上述规范。

在此我想稍稍抱怨一下(当然，我不行，我也不上，我实在就是想bb两句)，自从我开始研究JVM起，invokedynamic指令连同它所希望实现的动态语言支持简直就像是个毒瘤，JVM规范在面对这个问题时总会变得扭曲不自然。逻辑变得复杂难懂倒还在其次，最重要的还是失去了缜密的美感，让人看着难受。细思其缘由，我想还是因为该功能实在是与Java设计的初衷背离太远所致：需求分析的时候完全没在这地方留灵活度，上线后发现需求迫切没办法强行加功能。Java语言自最初起就被设计为了静态类型语言，但是随着时代的发展，人们对动态类型的需求逐渐迫切，Java也总是会因此被人指为不灵活，逐渐要被时代所抛弃。Java也在竭力进行改良，其结果就是现在我们看到的动态语言支持方案。个人认为这个方案比较糟(我没有更好的方案，但就是感觉比较糟)，其与Java基本的结构完全不同，仅仅只是一个为了实现功能的异类。好了=-=，抱怨结束。

invokedynamic指令所对应的引用被称为动态调用点限定符(Dynamic Call Site Specifier)，对于这条指令而言，其解析出的直接引用仅仅是本指令的本次调用生效，也就是说，必须要程序运行到这条指令，在当前的系统环境下，调用了才会知道真正的解析结果。

解析动作主要针对的是如下8种常量符号引用：

1. 字符串类型字面量(CONSTANT_String_info)
2. 类或接口的符号引用(CONSTANT_Class_info)
3. 字段的符号引用(CONSTANT_Fieldref_info)
4. 类中方法的符号引用(CONSTANT_Methodref_info)
5. 接口中方法的符号引用(CONSTANT_InterfaceMethodref_info)
6. 方法句柄(CONSTANT_MethodHandle_info)
7. 方法类型(CONSTANT_MethodType_info)
8. 动态方法调用点(CONSTANT_InvokeDynamic_info)

其中，1的解析过程非常简单，没什么可说的。6,7,8又是那个操蛋的动态语言支持系统，暂不研究。下面详细讲一下2~5。

**解析类或接口的符号引用**

假设字节码所处的类为D，若要把其常量池中的一个未解析过的符号引用N解析为一个类或接口C的直接引用，整个解析过程需以下3步：

1. 若C不是一个数组类型，那么JVM会将N所代表的全限定名传递给D的类加载器去加载C。该加载过程可能会触发新的类加载(例如需加载C的父类或C所实现的接口)，会进行必要的验证(元数据验证，字节码验证等)，一旦这整个流程的某一环节出现任何异常，解析即宣告失败。
2. 若C是一个数组类型，那么首先需找到该数组去掉所有维度信息后剩下的元素类型，然后按照第一点的规则去加载该元素类型。例如N为[Ljava/lang/Integer，则其元素类型即为java.lang.Integer。接着由JVM生成一个代表该数组的数组对象。
3. 若上述两步未产生异常，那么C在JVM中实际上已经是一个有效的，可被直接引用的类或接口了。即至此已确保了类或接口的存在性。随后只需要再进行符号引用验证，确认D是否具备对C的访问权限，若不具备权限，则抛出java.lang.IllegalAccessError。

**解析字段的符号引用**

字段的符号引用的结构如下图所示：

![2.jpg](/images/blog_pic/JVM/类加载机制/2.jpg)

从中先取出字段所属的类或接口的描述符，而后按照上文解析类或接口的符号引用的方法解析，若在该过程中发生任何异常，则本次字段的符号引用解析失败。若类或接口的解析成功，我们不妨仍将其命名为C，JVM会继续做如下操作：

1. 若C内部本身就包含本字段描述符对应的字段，则返回这个字段的直接引用，查找结束。
2. 否则，若C实现了接口，则按照实现顺序由前向后，在每个接口内部按照继承关系由下而上搜索是否包含本字段描述符，若找到则返回该直接引用，查找结束。
3. 否则，只要C不是java.lang.Object，均按继承关系从下而上递归搜索其父类是否包含本字段描述符，若找到则返回该直接引用，查找结束。
4. 否则，查找失败，抛出java.lang.NoSuchFieldError。

成功找到字段后，还需进行符号引用验证，确认是否具备对该字段的访问权限，若不具备权限，则抛出java.lang.IllegalAccessError。

按照上述搜索顺序，若某字段多次出现在C所实现的多个接口中，或是同时出现于C的父类或C所实现的接口中，是不会产生歧义的：按照规范总可有一个明确的搜索结果。然而javac编译器却要更为严格一些，上述情况其均会拒绝编译。这也很好理解，搞得这么乱，即便机器理得清，程序员也难免混乱。而且通常这种混乱都是编码问题造成的(名字而已，为啥非得重名不可)，算是Java语法在强行矫正程序员的不良编码习惯。举例如下：

```
public class Test {

    public static void main(String[] args) {
        System.out.println(C.A);
    }
}

interface I0 {

    int A = 0;
}

interface I1 {

    int A = 1;
}

class C implements I0, I1 {
    
}
```

该代码无法通过编译，在调用处(即C.A)会提示The field C.A is ambiguous。

**解析类中方法的符号引用**

类中方法的符号引用的结构如下图所示：

![3.jpg](/images/blog_pic/JVM/类加载机制/3.jpg)

其整体的解析思路与上文提到的字段的符号引用的解析基本一致。首先，从中先取出方法所属的类的描述符，而后按照上文解析类或接口的符号引用的方法解析，若在该过程中发生任何异常，则本次方法的符号引用解析失败。若类或接口的解析成功，我们不妨仍将其命名为C，JVM会继续做如下操作：

1. 若C是个接口，则解析失败并抛出java.lang.IncompatibleClassChangeError。
2. 在C内部查找符合方法描述符的方法，若找到则返回该方法的直接引用，查找结束。
3. 只要C不是java.lang.Object，均按继承关系从下而上递归搜索其父类是否包含待查找方法，若找到则返回该直接引用，查找结束。
4. 若C实现了接口，则按照实现顺序由前向后，在每个接口内部按照继承关系由下而上搜索是否包含待查找方法，若找到则说明C是抽象类，查找结束，抛出java.lang.AbstractMethodError。
5. 否则，查找失败。抛出java.lang.NoSuchMethodError。

成功找到方法后，还需进行符号引用验证，确认是否具备对该方法的访问权限，若不具备权限，则抛出java.lang.IllegalAccessError。

上述搜索过程会产生一些可能看起来不那么合理的结果，如下例：

```
public class Test {

    public static void main(String[] args) {
        new Son().m();
    }
}

interface I {

    void m();
}

class Parent {

    public void m() {
        System.out.println("m");
    }
}

class Son extends Parent implements I {
    
}
```

程序不仅可以编译通过，运行时也可正常输出：

```
m
```

Parent并未实现接口I，实现I的是Son。方法m实现于Parent，并未实现于Son。程序依然可以正常运行。

**解析接口中方法的符号引用**

接口中方法的符号引用的结构如下图所示：

![4.jpg](/images/blog_pic/JVM/类加载机制/4.jpg)

其整体的解析思路与上文提到的类中方法的符号引用的解析基本一致。首先，从中先取出方法所属的接口的描述符，而后按照上文解析类或接口的符号引用的方法解析，若在该过程中发生任何异常，则本次方法的符号引用解析失败。若类或接口的解析成功，我们不妨仍将其命名为C，JVM会继续做如下操作：

1. 若C是个类，则解析失败并抛出java.lang.IncompatibleClassChangeError。
2. 在C内部查找符合方法描述符的方法，若找到则返回该方法的直接引用，查找结束。
3. 按继承关系从下而上递归搜索C的父接口是否包含待查找方法，若找到则返回该直接引用，查找结束。
4. 否则，查找失败。抛出java.lang.NoSuchMethodError。

因为接口中的方法默认的访问权限均为public，因此无需进行符号引用验证。自然也不会抛出java.lang.IllegalAccessError。

# 初始化

初始化是类加载的最后一步。前面的几步是由JVM主导的，进行的也都是通用的分配空间，检查解析操作，几乎没有用户的参与(只有在加载阶段可能会调用用户自定义的类加载器)。而初始化阶段，一如其名，才是真正的按照用户定义的代码去初始化这个类。

在准备阶段，类变量已被赋过一次系统默认的零值(被final修饰的String或基本类型已赋上了最终值)，初始化阶段则是通过执行类构造器方法&lt;clinit&gt;来为其赋上用户自定义的值。

我们在编写Java源码时并未编写这个&lt;clinit&gt;方法，其实该方法是javac编译器在编译时收集所有类变量的赋值动作及static{}后按书写顺序自动生成的。但是其实也并非就是简单的将这些代码按顺序塞进&lt;clinit&gt;方法中。我们来看下面这个小例子：

```
public class Test {

    static {
        V = 1;    // 可以通过编译。即可以为在后面定义的类变量赋值
        System.out.println(V);    // 无法通过编译
                                  // 提示Cannot reference a field before it is defined
    }

    static int V;
}
```

另一个类似于类构造器方法&lt;clinit&gt;的是实例构造器方法&lt;init&gt;。二者都是由javac编译器自动生成的。二者有主要如下不同：

- 与前文已介绍过的&lt;clinit&gt;方法的收集内容及顺序不同的是，&lt;init&gt;方法先按源码编写顺序收集实例变量的赋值动作，而后收集构造函数中的信息。
- 构造函数的第一句话默认是调用父类构造函数(没显式指明的话则会默认调用父类的无参构造函数)，因此&lt;init&gt;方法总是会显式调用父类的构造函数，又因为构造函数可以重载，因此具体调用哪一个也未知。而&lt;clinit&gt;方法则不会显式调用其父类的类构造方法(只有一个，不会出现误解，没必要显式指明)，JVM总会确保某类的&lt;clinit&gt;方法在执行之前，其父类的&lt;clinit&gt;方法已执行完成。因此无论如何，JVM中第一个被执行的&lt;clinit&gt;方法必然属于java.lang.Object。

&lt;clinit&gt;方法对于类或接口而言并不是必须的，若该类或接口中既没有类变量的赋值动作，也没有static{}，那么javac则无需为其生成&lt;clinit&gt;方法。

与类不同的是，接口中不允许有static{}，但依然允许有类变量。因此接口依然可能会有&lt;clinit&gt;方法。只是除非用到了父接口的内容，接口在初始化时无需先运行其父接口的&lt;clinit&gt;方法。同样的，接口的实现类在初始化时除非用到了该接口中的内容，也无需初始化该接口。

JVM会自行解决并发环境下&lt;clinit&gt;方法的线程安全问题。即若有多个线程同时去初始化一个类，那么只有一个线程会真正的去执行&lt;clinit&gt;方法，其他线程都会被阻塞。执行&lt;clinit&gt;方法的线程执行完成退出后，因此时该类已加载完成(有一个细节需明确一下，JVM在判断内存中的类是否相同时既看类本身，也看加载该类的类加载器。即同样是对于类A，用类加载器1加载出的A同用类加载器2加载出的类A是两个不同的类。因此此处会保证所有线程都在用同一个类加载器加载这个类)，其他线程自然无需再次初始化，直接用即可。

因此在并发环境下，&lt;clinit&gt;方法可能会导致比较隐蔽的线程阻塞问题，有如下代码：

```
public class Test {

    static {
        System.out.println(Thread.currentThread().getName() + " init Test");
        while (true) {}
    }
}
```

很显然，这是想构造一个永不会完成的&lt;clinit&gt;方法。但显然javac编译器没那么笨，这样是无法通过编译的，会报Initializer does not complete normally。

不过javac也没那么聪明，略施小计即可令其懵逼：

```
public class Test {

    static {
        if (true) {
            System.out.println(Thread.currentThread().getName() + " init Test");
            while (true) {}
        }
    }
}
```

然后我们再稍加调整，并写出用于测试的main方法：

```
public class Test {

    public static void main(String[] args) {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " start");
                Loop loop = new Loop();
            }
        };
        new Thread(r, "Thread 1").start();
        new Thread(r, "Thread 2").start();
    }
}

class Loop {

    static {
        if (true) {
            System.out.println(Thread.currentThread().getName() + " init Test");
            while (true) {}
        }
    }
}
```

程序输出为：

```
Thread 1 start
Thread 2 start
Thread 1 init Test

```

该程序永不完结。