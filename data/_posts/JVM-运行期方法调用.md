---
title: JVM-运行期方法调用
date: 2017-12-06 17:55:49
tags: [Java,JVM,重载,重写]
categories: JVM
---

运行期方法调用的唯一目的就是确定被调用方法的版本(即确定调用哪个类的哪个方法)，执行方法体不是方法调用需考虑的问题。

<!-- more -->

对于C++这种编译执行的语言，方法的连接操作在编译结束后即完成。换句话说，编译结束后即可明确知道实际运行时该方法在内存布局中的地址。此时方法调用根本就不是一个问题：编译结果中写什么调用什么就好。

而Java这种以解释执行为主的语言将编译过程分为了两段：首先是.java源文件编译为.class的字节码文件，随后再是.class文件被解释执行或编译执行为本地机器码。这里.java到.class的转换虽然也被称为编译，却只是Java体系内部的转换，对于最终执行程序的本地机器而言，只要没有编译为它所认识的本地机器码，.java和.class对它而言都是一样的。因此.java到.class的过程其实并非真正意义上的编译，而只是一个中间过程。

不过.java到.class也确实是一种编译操作，因此我们仍然会称.java到.class的时期为Java编译期。只是务必在心里与类似C++那种真正的编译为机器码的行为做好区分。

.class文件的常量池集合中存储的方法地址均为符号引用。例如就是如下所示的字符串：

```
"<init>":()V
```

这种字符串易于人类理解阅读，也可用作标记，但却无法代表方法在内存中的入口。这是很容易理解的：Java体系中的编译期及运行期是被切割的，即可以编译好class文件后等很久再换一台机器执行。那么编译期自然不可能知道实际的内存布局是什么样子的，class文件中只能存储这样的符号引用。

这就会使得方法的连接操作(所谓连接，其实就是把class中记述的方法的符号引用替换为方法在内存中的实际入口地址，也就是Java中常说的直接引用)变得复杂，因为不管你Java有什么困难导致编译期无法确定方法的版本，只要你想完成最终的调用，那么在被最终转换为本地机器码之前必须完成方法的连接。既然编译期做不了，那么就只能在随后的运行期(只有两个时期，也没法推给其他人了)做了。运行期连接为Java带来了一定的动态扩展能力，却也相对的给方法的版本确认带来了麻烦，例如有如下调用逻辑：

```
G0 o = new G1();
// 一些代码...
o.m(P p);
```

其中G1为G0的孩子(儿子，孙子，重孙...)。而m又是一个可被子类继承的方法。那么在程序实际运行到o.m(P p);这一行之前，JVM是无法知道到底要调用哪个类的m方法的。虽然单看上例应该调用类G1，但是上例中被省略的"一些代码"可能是这样子的：

```
G0 o = new G1();
o = new G0();
o.m(P p);
```

此时该调用的类就是G0。那么在编译期javac编译到G0 o = new G1();时是无法知道后面会发生什么的。有人可能会说，o = new G0();就在它的下面，也在同一个类里，javac为什么识别不了呢？这个说的没毛病，理论上javac确实识别得了。但问题在于我么当然也可以不放在同一个类里：

```
G0 o = new G1();
X.m2(o);
o.m(P p);
```

X是与G(假设上述代码存在于类G中)完全无关的一个类，我们调用了它的m2方法并将o丢了进去。此时javac就真的无能为力了。因为类文件的编译都是以类为单位单独编译的，javac在编译G这个类时是无法知道X这个类的内容的，自然也不会知道X的m2方法中是否会有类似o = new G0();这样的操作。

这个问题对于运行期而言就有点严重了。因为在这个问题被摆到桌面上谈之前方法调用其实也不复杂：编译期其实可以确认方法该调用哪个版本，它所不知道的仅仅只是这个版本的方法在运行期被放到了内存的哪个位置而已。此时运行期需要做的操作仅仅只是把符号引用翻译为直接引用而已。但是当这个问题被抛出后，编译期无法确认方法该调用哪个版本，运行期在具体执行到o.m(P p);所代表的那个字节码指令之前自然也不可能知道。

这里所说的"不可能知道"其实有些绝对了，更确切的说法应该是"成本太高了，不想知道"。因为运行期的类加载过程其实基本上也是以类为单位加载的：需要一个加载一个。如果想在类加载阶段就确认这件事，那么相当于要将从G0 o = new G1();被声明，到o.m(P p);触发方法调用这期间的所有的"一些代码..."确认一遍，如果涉及到条件分支判断还要将所有分支可能导致的结果都记录下来。这个实现成本基本上就是在逼JVM开发人员自杀。

无奈之下JVM开发人员只好采取了折中的策略，说来说去可能会导致有问题的情况不也就上文说的那一种吗？那么我们将运行期的方法调用再分为两个阶段：静态多分派及动态单分派。

在具体介绍这两个阶段之前，需要先介绍一下这两个名字中那些被组合的单词。很显然，这和[Java 并发-同步异步阻塞非阻塞](/2017/10/03/Java 并发-同步异步阻塞非阻塞/)类似：同步异步是一组概念，阻塞非阻塞是一组概念，两组概念组合共能得到4个结果。同理，静态多分派及动态单分派中的静态动态是一组概念，多分派单分派是一组概念，两组概念组合依然能得到4个结果。不过这里，只有其中的两组是有意义的，也就是静态多分派及动态单分派。

所谓静态动态，指的就是能确定方法版本的时期，编译期能确定的就叫做静态，编译期确定不了的就叫做动态。

所谓单分派与多分派指的就是确定方法版本需要参考的参数(也叫做宗量)个数，如果只参考一个宗量就叫做单分派，需要参考多个宗量就叫做多分派。宗量这个概念并非出自JVM规范，而是源自《Java与模式》一书，不过这本书也足够经典，因此业界也就默认宗量这个概念的权威性了。单看分派，宗量这些概念仿佛很专业很复杂的样子。但其实在方法分配这件事上，宗量其实就只有两个：即"哪个类"的"哪个方法"。

对于静态多分派及动态单分派的具体含义后文将详述，不过在此我们可以先提纲挈领的总结一下。这两个概念看似让人很懵逼，但说穿了套路其实很简单，在分析之前我们需要再给出两个简单的小概念，即静态类型(Static Type)[也可称为外观类型(Apparent Type)]与实际类型(Actual Type):

```
G0 o = new G1();
o.m(P p);
```

以上代码中并没有烦人的"// 一些代码..."，因此我们可以确定当o.m(P p);执行时"哪个类"指的就是类G1。此时对于o而言，其静态类型为G0，实际类型为G1。

很显然，编译期能确定的是静态类型(Java是强类型语言，只要声明了,除非进行显式的强制类型转换，否则变量的类型就不能改变)，无法确定的是实际类型。

小概念介绍完毕，我们继续分析运行期确定方法版本的那两步：

第一步，所有方法都需要做，而且是就当编译期能确定所有方法的版本那样来做。"哪个类"统一认为是静态类型。发生于类加载的连接-解析阶段，因此也被称为解析(Resolution)。

第二步，执行了第一步后仍有问题的方法才需要做，发生于字节码执行引擎实际运行到方法调用的那行指令时。

那么什么方法才是"有问题的方法"呢？仔细分析上文中描述的问题就能发现，实际上这个问题本质上就是无法确认"哪个类"，而无法确认"哪个类"的原因则是该方法可能会被子类重写(Override)。至此我们已经找到了问题的答案："有问题的方法"指得就是那些能被子类所继承并重写的方法。换句话说，若一个方法无法被子类继承并重写，那么就只需要进行第一步。

思路其实很明确。编译期能确定的方法还按照当是没问题的方案来做，而对于那些可能会产生问题的方法则添加补救措施。注意这里说的仅仅是可能，因为允许被继承的方法并不代表它真的就会被继承，被继承了也不代表执行到方法调用那一行字节码指令时实际类型就一定是静态类型的子类型。不过，因为我们也无法保证这不可能，因此这些方法还是需要执行第二步。

# 静态多分派

如前文所述，只需执行第一步的方法是那些编译期可知，运行期不可变的方法，换句话说，也就是指那些不可被继承重写的方法(因为要变其实也就是通过继承的重写来变)。在Java中不可被继承的方法主要分为两大类：类方法(直接与本类静态类型所关联)及私有方法(子类不可访问)。

这里可以稍微讲一下方法的调用指令。[JVM-JVM字节码指令集](/2017/11/16/JVM-JVM字节码指令集/)中共提供了5条方法调用相关的指令：

- 0xb6(invokevirtual):调用实例方法。会根据对象的实际类型进行动态单分派(虚方法分派)
- 0xb7(invokespecial):以操作数栈栈顶reference类型的数据所指向的对象为方法的接收者，调用此对象的实例构造器&lt;init&gt;方法，私有方法或超类构造方法。该指令的操作码之后会紧跟一个u2的操作数说明具体调用的是哪个方法，该参数指向常量池集合中的一个CONSTANT_UTF8_info类型的索引项，也就是该方法的方法符号引用
- 0xb8(invokestatic):调用类方法(static修饰的方法)
- 0xb9(invokeinterface):调用接口方法。运行期解释器会搜索一个实现了该接口方法的对象，并调用对应的实现方法。

此外，还有那条操蛋的现在尚未出现于JVM字节码指令集中的invokedynamic，其会先在本指令运行期间动态解析出调用点限定符所引用的方法，然后再执行该方法。上文给出的那4条方法调用指令的方法分派逻辑是固化在JVM内部的，而invokedynamic的分派逻辑是由用户所设定的引导方法决定的。

根据前文分析，显然被invokestatic及invokespecial所调用的方法：静态方法，实例构造器&lt;init&gt;方法，私有方法，超类构造方法，都只需要执行第一阶段，因为它们都不可被继承。这些方法也被称为非虚方法。

但并不是说不是被invokestatic及invokespecial所调用的方法就是虚方法。因为被final修饰的方法是被invokevirtual指令所调用的，然而它虽然可以被继承，却无法被重写，不被重写就不会触发问题，因此它也是非虚方法，只需要执行第一步。除此之外，其他被invokevirtual指令调用的方法均是虚方法，可以被子类继承。需执行第二步。

这里需要再次明确的一点就是，静态多分派的方法版本是编译期就可确定的。注意这仅仅是可以确定，而不是真正的进行连接操作。无论如何(即便描述成静态的)，连接操作都是在运行期完成的。

因静态多分派阶段确认方法版本依据的是编译期的静态结果，因此在静态动态这组概念中属于"静态"，而其判定方法版本依据的是静态类型及方法签名(方法名+方法参数)，也就是既参考了"哪个类"，也参考了"哪个方法"，因此是单分派多分派这组概念中的多分派。

严格来说，分派(Dispatch)这个词在欧美一般是不用于静态环境中的。欧美那边对于第一步操作的称呼为Method Overload Resolution，即重载解决方案。这也道出了静态多分派阶段的难点：确定方法版本所用的两个宗量中静态类型是唯一不可变的。而方法签名中方法名则可能会相同，此时就会发生重载(这里依然是可能，重载并非是必然会发生的)。

关于方法签名有一个需要注意的点，静态多分派依据的信息全部来自于编译期，"哪个类"也完全认定为静态类型，这点对于方法签名中的参数类型也是一样的，即只会以方法参数的静态类型为准(还是那个问题，编译期是无法拿到实际类型的)：

```
public class Test { 

    public static void m(Parent parent) {
        System.out.println("parent");
    }

    public static void m(Son son) {
        System.out.println("son");
    }

    public static void main(String[] args) {
        Parent parent= new Son();
        Test.m(parent);
    }
}

class Parent {}

class Son extends Parent {}
```

输出：

```
parent
```

因此我们也可以这么说，完全依据静态类型来确定方法版本的分派方式为静态分派。

之所以上文说重载是静态多分派阶段的难点。是因为重载虽然在编译期就能完全确认方法版本，其确定依据却不是准确的。例如：

```
G.m(P p);
```

若G中的m存在重载方法，首先参数的个数及顺序是强制规范，这个是明确的。然而参数类型的判断却是模糊的。编译器会找方法形参中与实参静态类型血缘最接近的那个。例如，仍然是针对上面那条调用，若G中仅有一个类方法：

```
public static void m(P0 p0) {
}
```

其中P0是P的前辈(父亲，爷爷...)，那么按照血缘关系则定位到这个最亲近的方法。但若G中有如下两个类方法：

```
public static void m(P0 p0) {
}

public static void m(P p) {
}
```

此时依血缘最亲密的类需调用的方法就变成m(P p)。

对此，我们可以举一个比较极端的例子：

```
import java.io.Serializable;

public class SayHello {

    public static void sayHello(Object arg) {
        System.out.println("hello Object");
    }

    public static void sayHello(int arg) {
        System.out.println("hello int");
    }

    public static void sayHello(long arg) {
        System.out.println("hello long");
    }

    public static void sayHello(Character arg) {
        System.out.println("hello Character");
    }

    public static void sayHello(char arg) {
        System.out.println("hello char");
    }

    public static void sayHello(char ...arg) {
        System.out.println("hello char ...");
    }

    public static void sayHello(Serializable arg) {
        System.out.println("hello Serializable");
    }

    public static void main(String[] args) {
        SayHello.sayHello('a');
    }
}
```

输出：

```
hello char
```

没什么可说的，传入的为基本数据类char的值，自然精准重载到了对应方法上。此时我们删掉这个入参为char的方法，即变成：

```
import java.io.Serializable;

public class SayHello {

    public static void sayHello(Object arg) {
        System.out.println("hello Object");
    }

    public static void sayHello(int arg) {
        System.out.println("hello int");
    }

    public static void sayHello(long arg) {
        System.out.println("hello long");
    }

    public static void sayHello(Character arg) {
        System.out.println("hello Character");
    }

    public static void sayHello(char ...arg) {
        System.out.println("hello char ...");
    }

    public static void sayHello(Serializable arg) {
        System.out.println("hello Serializable");
    }

    public static void main(String[] args) {
        SayHello.sayHello('a');
    }
}
```

输出：

```
hello int
```

哦，看来是发生了一次自动的基本类型间的向上类型转换。因为'a'除了可代表字符a外，也可代表数字97(字符'a'在Unicode字符集中的编码就是97)。现在我们再把这个入参是int的方法去掉，变为：

```
import java.io.Serializable;

public class SayHello {

    public static void sayHello(Object arg) {
        System.out.println("hello Object");
    }

    public static void sayHello(long arg) {
        System.out.println("hello long");
    }

    public static void sayHello(Character arg) {
        System.out.println("hello Character");
    }

    public static void sayHello(char ...arg) {
        System.out.println("hello char ...");
    }

    public static void sayHello(Serializable arg) {
        System.out.println("hello Serializable");
    }

    public static void main(String[] args) {
        SayHello.sayHello('a');
    }
}
```

输出：

```
hello long
```

因为没找到int，因此再次向上转型为long。我没有写其他的基本数据类型，但显然转换可以沿着char->int->long->float->double这条转换链一直自动向上转换上去。当然，char是不会自动转型为未在它转换链上层的boolean，byte，short的。

现在我们再删掉这个入参是long的方法，变为：

```
import java.io.Serializable;

public class SayHello {

    public static void sayHello(Object arg) {
        System.out.println("hello Object");
    }

    public static void sayHello(Character arg) {
        System.out.println("hello Character");
    }

    public static void sayHello(char ...arg) {
        System.out.println("hello char ...");
    }

    public static void sayHello(Serializable arg) {
        System.out.println("hello Serializable");
    }

    public static void main(String[] args) {
        SayHello.sayHello('a');
    }
}
```

输出：

```
hello Character
```

此时已没有基本数据类型可供自动向上转型了，因此发生了一次自动装箱，'a'被自动装箱为其包装类Character。那么老套路，我们再删掉这个入参是Character的方法：

```
import java.io.Serializable;

public class SayHello {

    public static void sayHello(Object arg) {
        System.out.println("hello Object");
    }

    public static void sayHello(char ...arg) {
        System.out.println("hello char ...");
    }

    public static void sayHello(Serializable arg) {
        System.out.println("hello Serializable");
    }

    public static void main(String[] args) {
        SayHello.sayHello('a');
    }
}
```

输出：

```
hello Serializable
```

这个输出看起来就有些诡异了。不过我们可以看一下Character类的源码：

```
public final class Character implements java.io.Serializable, Comparable<Character> {
// 省略类内部代码
}
```

果然，Character实现了Serializable接口，看来编译器是认为此时血缘最亲近的就是它所实现的接口了。并且因为我们明明也写了入参是Character父类Object的方法而编译器却最终选择了Serializable。看来在编译器眼里，接口比父类要更亲密一些。不过我们不禁会产生另一个疑问：如果静态类型实现了多个接口(例如本例中的Character)，而又同时存在以这些接口为入参的重载方法，那么编译器会如何抉择呢？会按照接口在源码中的书写顺序来确定亲疏关系吗？有问题就要立刻验证，我们把代码修改如下：

```
import java.io.Serializable;

public class SayHello {

    public static void sayHello(Serializable arg) {
        System.out.println("hello Serializable");
    }

    public static void sayHello(Comparable<Character> arg) {
        System.out.println("hello Comparable<Character>");
    }

    public static void main(String[] args) {
        SayHello.sayHello('a');
    }
}
```

此时将无法通过编译，SayHello.sayHello('a');这一行会提示The method sayHello(Serializable) is ambiguous for the type SayHello。至此我不禁松了一口气，看来编译器还是有底线的。毕竟Java语言规范中可从来没有说过实现接口时接口的书写顺序会有什么优先级上的影响。

顺便再说一句，只有基本数据类型之间才可能发生自动类型转化。换句话说，是不能指望Character自动转换成Integer的。

然后我们再回到原来的代码上，现在我们再删掉入参是Serializable的方法：

```
public class SayHello {

    public static void sayHello(Object arg) {
        System.out.println("hello Object");
    }

    public static void sayHello(char ...arg) {
        System.out.println("hello char ...");
    }

    public static void main(String[] args) {
        SayHello.sayHello('a');
    }
}
```

输出：

```
hello Object
```

终于轮到父类了。事实上，编译器会在静态类型的继承链上依序向上搜索，最终追溯到Object类。这里需要注意的是，追溯时只看静态类型而不管实际存的是什么，极端的说，即便该静态类型指向的是null也无所谓。

然后我们再删掉这个入参是Object的方法：

```
public class SayHello {

    public static void sayHello(char ...arg) {
        System.out.println("hello char ...");
    }

    public static void main(String[] args) {
        SayHello.sayHello('a');
    }
}
```

输出：

```
hello char ...
```

此时就只剩下1个方法了，编译器也算是饥不择食了，最后连变长参数的方法也不放过。其实这也很好理解：咱们传入的参数被以组的概念理解了(只是这个组里只有1个元素)。不仅如此，变长参数方法还支持上文所有提到的那些关于亲密关系的判断。

至此这个例子便结束了。不得不说确实极端得可以，在这个例子中编译器简直是杨花水性，毫无节操。不过这种例子属于"茴香豆的茴有几种写法"一类的问题，虽然确实也是个知识点，这么写也确实不会报错，然而实际开发中完全是然并卵的，因为基本不会有什么业务需求会抽风到逼程序员写出这样的代码，而且要是真有程序员写出这种代码他也基本会被他的同事砍死。它和Java中的i++,++i一类问题一样，往往都只会以考校程序员功底的目的出现在面试或期末考试中。

# 动态单分派

关于动态单分派，我们先解释一下名称。因为第二步发生在字节码执行引擎执行到方法调用那条字节码指令时，依据的是程序在运行期实际的动态运行结果而非编译期得出的静态结果，因此在静态动态这组概念中是动态。又因为第二步诞生的目的就是为了解决实际类型可能与静态类型不一样的问题，也就是说到了第二步，决定方法调用版本中的"哪个方法"问题在第一步中已能完全确定了，可能有问题的仅仅是"哪个类"，因此判断方法版本所需依据的宗量仅仅只有实际类型一个。因此在单分派及多分派这组概念中为单分派。

这一步实际上就是在解决上文中提到的那个因重写造成的问题的。当字节码执行引擎执行到具体的调用指令时，自然可以知道"哪个类"的实际类型是什么。

关于动态单分派，如果完全遵循上文所描述的概念模型。那么每次在确认某父类中的可继承方法是否真的被子类重写时，都需要重新搜索一遍父类和子类。而动态单分派的执行频率又很高，这就有些浪费资源了：因为编译期虽然无法确定实际类型是什么，但是确定方法的继承关系则毫无难度。因此虽然JVM实现间千差万别，但大多都会进行如下"稳定优化"手段：为类在方法区中建立一张虚方法表(Virtual Method Table，简称vtable)。当需要进行动态单分派时，就可以查找这张虚方法表而非重复搜索父类和子类了。

例如有如下代码：

```
class QQ {}

class _360 {}

class Father {

    public void hardChoice(QQ arg) {
        System.out.println("father choose qq");
    }

    public void hardChoice(_360 arg) {
        System.out.println("father choose 360");
    }
}

class Son extends Father {

    @Override
    public void hardChoice(QQ arg) {
        System.out.println("son choose qq");
    }

    @Override
    public void hardChoice(_360 arg) {
        System.out.println("son choose 360");
    }
}
```

这段代码中Father与Son的虚方法表如下图所示：

![0.png](/images/blog_pic/JVM/运行期方法调用/0.png)

虚方法表创建于运行期类加载的连接-解析阶段，它诞生于运行时数据区域之一的方法区中，而非编译期产出的class文件中。因此虚方法表中存储的已经可以是各实际类型的方法指向方法区的直接引用，是拿来直接就能用的，而非class文件中那种符号引用。如果父类的某个方法子类没有重写，那么子类虚方法表中该方法的实际引用地址与其父类虚方法表中该方法的实际引用地址相同，都指向父类的方法。若子类重写了父类的某方法，则子类虚方法表中该方法的实际引用将指向子类重写后的方法。上图中，Son重写了Father中所有的方法，因此Son的虚方法表中没有指向Father虚方法表的箭头。但是Father没有重写其父类Object中的方法，因此对这些方法而言，Father的箭头均指向Object。同理，Son依然没有重写这些方法，因此它会将这些方法的箭头指向Father，进而又被指向Object。

为了程序实现上的方便，具有相同签名的方法，在父类，子类的虚方法表中的索引号应该保持一致。这样当实际类型变化时(也就是"哪个类"变化时)，只需要变更虚方法表，而无需变更索引号(即"哪个方法"不变)，这也与动态单分派中单宗量的逻辑吻合。

与虚方法表类似的，往往也会为类在方法区中建立一张invokeinterface指令会用到的接口方法表(Inteface Method Table，简称itable)。

虚方法表是大多JVM实现都会采用的"稳定优化手段"。在条件允许的情况下，JVM实现也可能会采用另外一些激进的，同样是发生于晚期(运行期)的"非稳定优化手段"：

- 内联缓存(Inline Cache)
- 基于类型继承关系分析(Class Hierarchy Analysis,CHA)技术的守护内联(Guarded Inlining)