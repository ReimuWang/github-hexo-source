---
title: Java 设计模式-9.Bridge模式
date: 2018-07-13 10:44:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Bridge模式被归入了第4部分[分开考虑]()。在GoF原书中，Abstract Factory模式则被归入了[创建型设计模式]()。简单来说，Bridge模式可以被描述为：将类的功能层次结构与实现层次结构分离。

<!-- more -->

# 综述

Bridge是桥梁中的意思，就像在现实生活中，桥梁的作用是将河两岸的土地连接在一起那样，Bridge模式的作用也是将两样东西连接在一起，这两样东西是：

- 类的功能层次结构
- 类的实现层次结构

在讲解Bridge模式之前，有必要先介绍下这两种层次结构。

# 类的两种层次结构

**类的功能层次结构**

假设我们现在有名为Something的类，我们希望在不破坏Something的结构，即不修改Something代码的前提下基于Something类扩展出新的功能，或者更具体的说，希望能加入一个新的方法。那么最常见的做法自然就是写一个子类继承Something，比如名为SomethingGood，并将新的方法添加到SomethingGood中。当然如果我们想继续扩展，比如在SomethingGood的基础上再加点什么，那么我们可以再写一个SomethingGood的子类SomethingBetter，依次类推下去。当然，为了维护和理解的便利性，这种继承的层级不应过深。

这就是为了增加新功能而产生的层级结构，即：父类中具备基本功能，然后在子类中扩展新的功能。通常，我们会将这种层级结构称为"类的功能层次结构"。

**类的实现层次结构**

在Template Method模式中，父类定义了整体的流程，而具体的实现则没有定义，它们会以抽象方法的形式被声明，进而约束子类必须去实现。这种层级结构的设计目的显然就不是为了添加新功能了，反过来说，在这种层级结构中添加新功能会非常麻烦，因为要在父类中添加一个新的抽象方法，所有已有的子类都必须修改一遍。

那么这种层级结构的优势是什么呢？很显然，添加一个新的实现是非常容易的：此时无需修改任何已有代码，只要按照父类的约束生成新的子类即可。我们说Abstract Factory模式"易于扩展具体的工厂，难以增加新的零件"其实也是一样的道理。

这种层级结构就被称为"类的实现层次结构"。

反之，显然，类的功能层次结构虽然易于增加新的功能，却难以添加新的实现：因为新功能下沉到了下面的层级，导致上层的功能并不丰富，在我们想添加新的实现时，可能不得不添加重复的代码或者做较大的代码调整：例如将已沉在下层的功能提升到上层。

**两种层次结构的混合使用**

通过上文的分析，我们知道了这两种层次结构是不相容的：一个的优势就是另一个的劣势。但是我们的日常需求大多都不是单一的：绝大多数时候，我们会既希望增加新功能，又增加新实现。

Bridge模式就是为了解决这一矛盾而诞生的：它会将类的功能层次结构与类的实现层次结构分离，并在二者之间搭建桥梁。即在保证实现需求的前提下，优化代码层级关系。

# 示例程序

下面我们给出一个应用Bridge模式的小例子，该例子的作用是"显示一些东西"。这么说似乎有些抽象，不过具体看例子就很容易理解了。

首先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/9Bridge模式/0.jpg)

本程序中的所有代码将被统一置于design9包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/9Bridge模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

首先是类的功能层次结构：

**Display类**

```
package design9;

public class Display {

    protected DisplayImpl impl;

    public Display(DisplayImpl impl) {
        this.impl = impl;
    }

    public final void display() {
        this.impl.open();
        this.impl.print();
        this.impl.close();
    }
}
```

**CountDisplay类**

```
package design9;

public class CountDisplay extends Display {

    public CountDisplay(DisplayImpl impl) {
        super(impl);
    }

    public void multiDisplay(int times) {
        this.impl.open();
        for(int i = 0; i < times; i++) this.impl.print();
        this.impl.close();
    }
}
```

然后是类的实现层次结构：

**DisplayImpl类**

```
package design9;

public abstract class DisplayImpl {

    abstract void open();

    abstract void print();

    abstract void close();
}
```

**StringDisplayImpl类**

```
package design9;

public class StringDisplayImpl extends DisplayImpl {

    private String str;

    public StringDisplayImpl(String str) {
        this.str = str;
    }

    @Override
    void open() {
        this.printLine();
    }

    @Override
    void print() {
        String ch = "|";
        System.out.println(ch + this.str + ch);
    }

    @Override
    void close() {
        this.printLine();
    }

    private void printLine() {
        String ch = "+";
        System.out.print(ch);
        for (int i = 0; i < this.str.length(); i++)
            System.out.print("-");
        System.out.println(ch);
    }
}
```

最后是测试类：

**Main类**

```
package design9;

public class Main {

    public static void main(String[] args) {
        DisplayImpl displayImpl = new StringDisplayImpl("ReimuWang");
        Display display = new Display(displayImpl);
        display.display();
    }
}
```

运行后输出：

```
+---------+
|ReimuWang|
+---------+
```

如果我们将代码变为：

```
package design9;

public class Main {

    public static void main(String[] args) {
        DisplayImpl displayImpl = new StringDisplayImpl("ReimuWang");
        Display display = new CountDisplay(displayImpl);
        display.display();
    }
}
```

再次运行后，结果显然是不会变的。

如果我们将代码变为：

```
package design9;

public class Main {

    public static void main(String[] args) {
        DisplayImpl displayImpl = new StringDisplayImpl("ReimuWang");
        CountDisplay display = new CountDisplay(displayImpl);
        display.multiDisplay(5);
    }
}
```

此时输出为：

```
+---------+
|ReimuWang|
|ReimuWang|
|ReimuWang|
|ReimuWang|
|ReimuWang|
+---------+
```

# 关于示例程序的说明

继承是强关联，例如示例程序中的Display与CountDisplay，DisplayImpl与StringDisplayImpl。

委托是弱关联，例如示例程序中的Display与DisplayImpl。对于调用者而言，他们只知道Display，而实际上，在Display内部，所有具体的工作都被委托给了DisplayImpl。

而Bridge模式的桥梁正是通过这种委托实现的。这有些类似于Builder模式(仅仅只是类似，其实还是有不小差别的)：功能层次结构这一侧类似于设计者，负责对外提供功能，而实现层次结构这一侧则类似于建造者，负责具体的细节。

这样，正像例子中演示的那样。如果我们想添加一个新的功能，我们就可以在功能层次结构这一侧定义一个子类CountDisplay，然后将我们需要添加的新功能作为一个方法multiDisplay写进去。因为实现方法的"小积木"在实现层次结构这一侧都已经做好了，我们只需要拼接出一个新的功能即可。这样本次改动仅仅就只是在功能层次结构这一侧增添了一个新的子类，而无需任何其他改动。

当然，这样说其实是默认了一个前提，就是构成新功能的"小积木"在实现层次结构都已经做好(这也是给实现层次结构提出的要求，所有功能都要尽量小，尽量单一，这样复用性才高)，但事实上，无论如何做准备，也无法预知到未来所需的一切功能：总会有"小积木"没有做好的时候，此时的改动量就要大一些了。因为除了要在功能层次结构这边增加一个子类之外，还需在实现层次结构那边的所有实现中补上欠缺的"小积木"。相对而言，在实现层次结构中增加新的实现就要简单很多了：无论如何，我们都只需要增加一个新的实现即可。

# 登场角色

上面的示例程序介绍了Bridge模式的Java实现，下面咱们试着跳出语言层面，抽象出Bridge模式中登场的角色。

**FunAbstract(功能层次结构中的父类)**

示例中，由Display类扮演这个角色。

**FunConcrete(功能层次结构中的子类)**

示例中，由CountDisplay类扮演这个角色。

**ImplAbstract(实现层次结构中的父类)**

示例中，由DisplayImpl类扮演这个角色。

**ImplConcrete(实现层次结构中的子类)**

示例中，由StringDisplayImpl类扮演这个角色。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/9Bridge模式/2.jpg)

# 相关设计模式

**[7.Builder模式]()**

正如前文所分析的，二者有很多相似之处。

---

**[13.Visitor模式]()**

二者的核心思想均为"分离"。