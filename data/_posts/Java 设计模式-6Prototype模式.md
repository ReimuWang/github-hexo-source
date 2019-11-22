---
title: Java 设计模式-6.Prototype模式
date: 2018-06-25 11:16:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Prototype模式被归入了第3部分[生成实例]()。在GoF原书中，Prototype模式则被归入了[创建型设计模式]()。简单来说，Prototype模式可以被描述为：复制已有实例以生成新的实例。

<!-- more -->

# 综述

在Java中，如果我们想要获得一个实例，最通常的做法就是找到这个实例所属的类，然后通过new关键字生成。这种基于模版生成实例的做法最符合思维逻辑，但某些情境下却不大方便。

下面我们就来举一个这种"不大方便"的小例子吧。

假设我们用Java写了一个画图程序，然后我们用该程序画了一个圆。站在编程的角度考虑，我们画出的圆是实例，程序中会有一个名为"圆"的类作为模版来生成它。要描述一个圆，假定需要如下参数：

- 圆心位置
- 半径
- 是否填充颜色，如果是，填充的颜色是什么
- 边线粗细(如果为0代表无边线)
- 边线颜色
- 边线是否为虚线，如果是，虚线的样式是什么样的？

当我们第一次绘制这个圆的时候，自然是要用new关键字基于模版创建的。但如果我们要复制这个实例呢？

比较容易想到的方式自然就是再用相同的参数new一个新的圆啦！不过这样就需要我们把描述这个实例的参数都记录下来，或者是能做到从待复制实例中提取出来。在参数较多的时候，比如我们要复制的不是一个圆，而是用户绘制的一幅复杂的图画，这样的设计既繁琐又不利于维护。

不过话说回来，既然我们都打算把属性从实例中提取出来了，那么为什么不在复制时跳过模版这一层，直接复制实例以得到新的实例呢？我们当然可以这样做，由这种想法延伸出来的设计模式就是Prototype模式。

Prototype的含义是"原型"，也就是待复制的实例。我们可以这样来理解：传统的基于类生成实例的方式就好比打印机，它需要切实了解文档的内容，这样才能从无到有的打印出文档。而Prototype模式则好比是复印机，首先我们需要拿到一份文档实例，如果我们想复印它，那么我们并不需要了解它的内容，也不需要知道它是如何生成的，我们只需要原样复印即可。

当然，使用复印机必须先获得待复印的文档，而这第一份文档还是需要打印机生成的。上文中关于"圆"的那个例子也是这样：第一个圆的实例还是需要通过new关键字来生成。

幸运的是，Java API为我们提供了Cloneable接口，直接使用即可实现实例的复制。这里多说一句，Java API提供的是浅克隆(shallow copy)，也被叫做字段对字段的复制(field-to-field-copy)。即只会克隆一层。例如我们有对象a，a中有B类型字段b。假设我们通过克隆生成了对象a1，那么a1中的类型为B的字段已经是复制后的了，我们可以将它命名为b1，它与a中的b是两个不同的引用。不过二者指向的实体却没有变。这件事的道理和Java中的值传递与址传递类似(Java中只有值传递哦)，算是老生常谈的问题了。

如果我们想要实现N层拷贝，或者是真正意义上的深拷贝，那就只能重写clone()方法自己实现了。

另外，需要注意的是，Java API提供的复制只是基于原实例中的字段生成新的实例，并不会调用构造函数。

# 示例程序

下面我们给出一个更具体的小例子，来加深一下对Prototype模式的理解。该示例程序的功能为给字符串加上修饰符，修饰符可能是环绕文字的符号，也有可能是下划线。

首先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/6Prototype模式/0.jpg)

本程序中的所有代码将被统一置于design6包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/6Prototype模式/1.jpg)

其中Initialize.java及Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Product接口**

```
package design6;

public interface Product extends Cloneable {

    /**
     * 具体的打印逻辑
     */
    void use(String str);

    /**
     * 复制自身
     */
    Product createClone() throws CloneNotSupportedException;
}
```

**Manager类**

```
package design6;

import java.util.HashMap;
import java.util.Map;

public class Manager {

    private Map<String, Product> showcase = new HashMap<String, Product>();

    public void register(String name, Product proto) {
        this.showcase.put(name, proto);
    }

    public Product create(String name) throws CloneNotSupportedException {
        return this.showcase.get(name).createClone();
    }
}
```

**MessageBox类**

```
package design6;

/**
 * 在字符串周围加上环绕字符
 */
public class MessageBox implements Product {

    /**
     * 用于环绕字符串的修饰字符
     */
    private char decochar;

    /**
     * 字符串距左右修饰字符的间隙
     */
    private int padding;

    public MessageBox(char decochar, int padding) {
        this.decochar = decochar;
        this.padding = padding;
    }

    @Override
    public void use(String str) {
        this.printLine(str);
        System.out.print(this.decochar);
        for (int i = 0; i < this.padding; i++)
            System.out.print(" ");
        System.out.print(str);
        for (int i = 0; i < this.padding; i++)
            System.out.print(" ");
        System.out.print(this.decochar);
        System.out.println();
        this.printLine(str);
    }

    @Override
    public Product createClone() throws CloneNotSupportedException {
        return (MessageBox)this.clone();
    }

    private void printLine(String str) {
        for (int i = 0; i < str.length() + (this.padding + 1) * 2; i++)
            System.out.print(decochar);
        System.out.println("");
    }
}
```

**UnderlinePen类**

```
package design6;

/**
 * 在字符串下方加上下划线
 */
public class UnderlinePen implements Product {

    /**
     * 组成下划线的字符
     */
    private char ulchar;

    public UnderlinePen(char ulchar) {
        this.ulchar = ulchar;
    }

    @Override
    public void use(String str) {
        System.out.println(str);
        for (int i = 0; i < str.length(); i++)
            System.out.print(this.ulchar);
        System.out.println();
    }

    @Override
    public Product createClone() throws CloneNotSupportedException {
        return (UnderlinePen)this.clone();
    }
}
```

**Initialize类**

```
package design6;

public class Initialize {

    public static Manager initialize() {
        Manager manager = new Manager();
        manager.register("big box", new MessageBox('$', 5));
        manager.register("small box", new MessageBox('=', 2));
        manager.register("strong underLine", new UnderlinePen('*'));
        manager.register("weak underLine", new UnderlinePen('-'));
        return manager;
    }
}
```

其实Initialize与Main都是测试代码，和Prototype模式本身无关。之所以分成两个类是为了表达以下含义：原型的生成通常只会在系统初始化时进行一次。而原型的复制则是在每次需要时进行。

**Main类**

首先测试一下"大盒子"：

```
package design6;

public class Main {

    public static void main(String[] args) throws CloneNotSupportedException {
        Manager manager = Initialize.initialize();
        Product bigBox = manager.create("big box");
        bigBox.use("ReimuWang");
    }
}
```

输出：

```
$$$$$$$$$$$$$$$$$$$$$
$     ReimuWang     $
$$$$$$$$$$$$$$$$$$$$$
```

稍微改造一下，再来看看"小盒子"：

```
package design6;

public class Main {

    public static void main(String[] args) throws CloneNotSupportedException {
        Manager manager = Initialize.initialize();
        Product smallBox = manager.create("small box");
        smallBox.use("ReimuWang");
    }
}
```

输出：

```
===============
=  ReimuWang  =
===============
```

然后是强下划线：

```
package design6;

public class Main {

    public static void main(String[] args) throws CloneNotSupportedException {
        Manager manager = Initialize.initialize();
        Product strongUnderLine = manager.create("strong underLine");
        strongUnderLine.use("ReimuWang");
    }
}
```

输出：

```
ReimuWang
*********
```

最后是弱下划线：

```
package design6;

public class Main {

    public static void main(String[] args) throws CloneNotSupportedException {
        Manager manager = Initialize.initialize();
        Product weakUnderLine = manager.create("weak underLine");
        weakUnderLine.use("ReimuWang");
    }
}
```

输出：

```
ReimuWang
---------
```

# 针对示例程序的说明

**原型与单例**

在Prototype模式中，作为原型的那个实例自然是唯一的。但并不意味着原型实例所属的那个类就要是单例的，因为同一个类可能会产生多个原型(例如上文示例中的big box与small box)，而且我们的目的就是对原型进行复制。

如果必要，我们还会修改复制出的结果。这点在上例中并不明显。因为我们都是拿到原型的克隆体后直接就用了。但是实际上，原型可以被理解为一个默认值，很多时候我们在拿到克隆体后是需要修改的。那么有人可能要问了，既然都要改了为什么不直接生成一个新的呢？那是因为我们需要的实例还是基于原型生成的，绝大多数的属性都与原型类似，因此在原型的基础上修改还是比生成一个新的实例容易。例如最开始的那个"圆"的例子，当我们拿到复制的圆后，也可以微调圆的形状。

进一步来说，原型到底是个什么东西呢？

我认为，原型是介于类与实例之间的一个东西。以上例的big box及small box为例，它们有一定的通用性，我们通常会希望像使用样式模版那样使用它们。但是它们又不足以作为一个类：毕竟它们仅仅只是一个样式而已，如果我们将这些样式都设计为类，那么系统中的类就会太多了。例子中是只设计了两种盒子，但是我们可以通过简单的参数调整就设计出无数个盒子：大大盒子，小小盒子等等。显然这是不利于维护的。

**使用与具体的实现解耦**

严格来说，这个特性并不是Prototype模式的核心特性。Prototype模式的核心特性其实就是"直接复制原型生成新的实例"。只不过，原型作为一种实例，量还是比较大的。因此对原型的管理也是很现实的问题。

首先，来看Product与 UnderlinePen/MessageBox。显然，如果原型逻辑比较简单，仅仅就是需要一种大的样式：例如只需要环绕字符串或者加下划线，也能够确定后续不会再扩展什么别的处理样式，那么完全可以不用设计这层抽象的Product。不过正如我们下文要论述的，这样不利于框架与实现的解耦，而且"确定后续不会再扩展什么别的处理样式"云云说得确实也太绝对了，没人能准确的预知未来的事情，因此花费少许的代价(多定义一层抽象的结构)，换得程序架构的清晰及后续的可扩展性还是很值得的。

然后观察上例中的Main.java及Initialize.java，我们可以发现两个特点：

第一，在Main中做调用时，只出现了作为框架的类Manager及Product，没有出现任何具体的实现类。显然，这样可以做到"高内聚，低耦合"，让使用与生成解耦。

第二，进一步的，Main中的使用者不仅不需要使用具体的实现类，他们甚至都不需要知道实现类到底是如何组织的。对于他们而言，他们只知道系统提供了四种修饰字符串的样式：

- big box
- small box
- strong underLine
- weak underLine

虽然从底层实现的角度来说，big box及small box这两种原型是基于MessageBox类生成的，而strong underLine及weak underLine这两种原型是基于UnderlinePen类生成的。但是如果使用者不需要知道这些细节，他们完全可以把这四种样式视为并列独立的，只管按名字调用即可。

也就是说，虽然系统提供的是原型，但使用者在使用时却仿佛在使用类。

# 登场角色

上面的示例程序介绍了Prototype模式的Java实现，下面咱们试着跳出语言层面，抽象出Prototype模式中登场的角色。

**Prototype(原型)**

Prototype提供了生成原型，复制原型以及对外提供克隆体的能力。在示例程序中，它首先被一分为二：管理原型及对外提供原型克隆体的功能被放入了Manager类中。而Product/UnderlinePen/MessageBox这一系则负责Prototype的核心功能：即原型的提供及复制。在这一系的内部，Product接口负责扮演抽象的框架的那一层(当然，不仅仅是接口，依需求不同，抽象类，甚至是非抽象类，都可以扮演这一层的角色)， UnderlinePen/MessageBox 类负责扮演具体实现的那一层。

可能有人会觉得Prototype模式的角色有些少，怎么会只有一个呢？但是从本质上来讲，Prototype模式的角色真的就只有Prototype这一个，示例程序中出现的其他的所有角色不过是为了让程序更好而做的补充。它们并不是Prototype模式所关注的重点。而且这些外围角色所承载的功能是需要依需求的不同灵活变化的。例如示例程序中Manager这个类，它只管理了Product这一系的原型。如果我们需要定义完全不同的另一系原型，那么我们依然可以将它们交由Manager管理。

这其实给了我们很好的启示：设计模式并不是孤立的，也并非一成不变的。说实话，当我们使用某个设计模式时，使用方式是否规范并不重要(话说回来，设计模式本来就是经验的总结，其实没什么规范可言)，基于需求做出最合理的应对才是最重要的。这也就是所谓的"黑猫白猫，抓到老鼠就是好猫"。因此，在研究设计模式时，应尽可能的探究它的理论内核，而不应被一些表象上的条条框框限制住。这样才能在需要时做出最佳的变通及组合。

进一步来说，我们研究设计模式，虽然明面上有什么23种GoF设计模式这一说，但这23种设计模式其实仅仅是供我们探究设计模式的跳板。当我们研习完这23种设计模式，如果仅仅是把它们背了下来，意义其实并不大(可能也就面试的时候忽悠一下面试官吧)，关键是透过现象看本质，找到这些设计模式中共通的"思想"。一旦领悟到这一层，其实就没有什么特定的设计模式可言了。基于需求，我们可以很自然的将几种设计模式组合变形，形成最适用当前环境的新的设计模式。

在[倚天屠龙记]中，张三丰在教授张无忌太极剑法时，曾反复问张无忌忘了多少剑招了，直到张无忌说全忘记了，老道才在众人的一片懵逼中笑道："不坏不坏，忘得真快"。因为剑招其实都是次要的，剑意才是最重要的。如果硬要从技术的角度分析，太极剑法只有一招:那就是不断的画圈，大圈套小圈，正圈套斜圈。但是1招中却又蕴含着无穷多招，正所谓太极圆缺，无使断绝，无招胜有招。

说了这么多题外话，我想表述的就是在学习设计模式时，具体的设计模式都是招式，我们学会了可以快速上手运用，但是更重要的还是通过这些招式领会背后的"思想"。一旦融汇贯通，其实也就没什么23种设计模式可言了。对我们而言，设计模式其实只有一种，那就是"基于当前环境，生成的那种最恰当的设计模式"。而因为我们可能面对的环境有无穷多种，我们可用的设计模式自然也无穷无尽。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/6Prototype模式/2.jpg)