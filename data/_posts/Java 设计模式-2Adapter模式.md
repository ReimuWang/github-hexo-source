---
title: Java 设计模式-2.Adapter模式
date: 2018-06-08 10:27:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Adapter模式被归入了第1部分[适应设计模式]()。在GoF原书中，Adapter模式则被归入了[结构型设计模式]()。简单来说，Adapter模式可以被描述为：连接具有不同接口(API)的类。也就是所谓的加个"适配器"以便于复用。

<!-- more -->

# 综述

Adapter这个单词的释义为"使...相互适应的东西"。在计算机领域，它被翻译为"适配器"。

其实不仅仅是计算机领域，适配器在我们的生活中是很常见的东西。例如老式的电脑只能以VGA接口连接显示器，而时下的显示器很有可能已经取消VGA这个过时的接口了，而只有HDMI和DI接口。这种情况下电脑与显示器是无法连接的，换掉其中一方以适配又很浪费(毕竟它们都还是挺贵的)。这个时候相对来说便宜很多的"转接头"(这是通俗的叫法，其实就是适配器)就派上用场了：例如VGA-HDMI转接口，顾名思义，它的一端连接VAG接口，另一端连接HDMI接口。这样不同接口的电脑与显示器就可以相互连接了。

在程序世界中，适配器的作用大抵也是如此。例如我们经常会遇到现有的程序无法直接使用，需要做适当的变换之后才能使用的情况。这种用于填补"现有的程序"与"所需的程序"之间差异的设计模式就是Adapter模式。

Adapter模式也被称作Wrapper模式。Wrapper的含义是"包装器"。就像用精美的包装纸将普通商品包装成礼物那样，Wrapper会替我们把某样东西包起来，使其能够用于其他用途。

Adapter模式有以下两种具体的实现：

模式一：类适配器模式(使用继承的适配器)。

模式二：对象适配器模式(使用委托的适配器)。

本文将依次介绍这两种模式。

# 模式一：类适配器模式(使用继承的适配器)示例程序

首先，给出给出本程序的类图：

![0.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/0.jpg)

在介绍程序之前，我们不妨再更详细的分析一下本文开篇时提到的"显示器连接电脑"的问题。在这个问题中，一共出现了4个角色：

1. 电脑：待显示内容的提供者。负责提供数据。
2. 显示器：与人类交互的接口。也就是说，显示器显示了一个画面，这个画面在电脑内部肯定是以某种形式存储的，然后显示器解析这个数据并将之转换为了易于人类理解的画面。显示器不实际生产数据，它只是向人类展现数据。对于人类而言，在显示画面这个需求上，他是不需要知道电脑的存在的，他只需要看显示器提供的画面即可。
3. 适配器：如果电脑与显示器不是适配的，即电脑传输的数据显示器无法解析，那么就需要适配器将该数据转换为显示器能理解的格式。
4. 人类：观看显示器展现的画面。

类似的，本程序也有4个一一对应的角色：

1. Banner类：对应于电脑。负责实际的功能。它内部包含两个方法。其中showWithParen()方法会为字符串加上()。showWithAster()方法会为字符串加上**。
2. Print接口：对应于显示器。不实际生产数据。但却是本程序中负责与观察者(即人类，或main函数)交互的组件。它内部包含两个方法。其中printWeak()方法会弱化字符串。printStrong()方法会强调字符串。
3. PrintBanner类：对应于适配器。
4. Main类：对应于观看屏幕的人。其中包含main方法，是程序的入口。会对本程序的功能进行调用。

总结来说，就是Main类使用PrintBanner类以期实现Print接口所约束的功能。而该功能在PrintBanner类内部实际是通过Banner类实现的。

本程序中的所有代码将被统一置于design2包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/1.jpg)

下面就来逐一看下具体的代码吧。

**Banner类**

```
package design2;

public class Banner {

    private String str;

    public Banner(String str) {
        this.str = str;
    }

    public void showWithParen() {
        System.out.println("(" + this.str + ")");
    }

    public void showWithAster() {
        System.out.println("*" + this.str + "*");
    }
}
```

**Print接口**

```
package design2;

public interface Print {

    void printWeak();

    void printStrong();
}
```

**PrintBanner类**

```
package design2;

public class PrintBanner extends Banner implements Print {

    public PrintBanner(String str) {
        super(str);
    }

    @Override
    public void printWeak() {
        super.showWithParen();
    }

    @Override
    public void printStrong() {
        super.showWithAster();
    }
}
```

这里适配器直接调用了Banner类的方法。实际上，两个待适配的组件是不会匹配得这么好的(否则也不用适配了)，内容还是那些内容，不过适配器通常都要做一些组合和转换。

**Main类**

```
package design2;

public class Main {

    public static void main(String[] args) {
        Print print = new PrintBanner("ReimuWang");
        print.printStrong();
        print.printWeak();
    }
}
```

运行Main.java后输出：

```
*ReimuWang*
(ReimuWang)
```

本例中，为了简单起见，我们将PrintBanner类的声明放到了main函数中。但事实上，调用者是完全不需要知道PrintBanner的。对于使用者而言，它只管拿到一个实例，这个实例实现了Print接口，多数情况下，这个实例都是外部传递给调用者的，它根本不用操心这个实例是怎么来的，内部实现是什么样子的，它就只需要利用这个实例来调用Print接口约束的功能就可以了。

# 模式二：对象适配器模式(使用委托的适配器)示例程序

在介绍对象适配器模式之前，有必要先简要介绍一下委托。

简单来说，委托就是"交给其他人"。比如我们需要去领取一份材料，但恰好没有时间。此时我们就可以委托他人代为领取。当然，如果这份材料足够重要的话(例如新办理好的身份证)，还需要我们出具一份委托书，大意就是"本人委托xx代我领取xx材料"。在编程语言中，委托的含义也是大抵如此，就是指将某个方法中的实际业务处理交付给另一个方法。

所谓适配器，最重要的一个特征就是要能同时兼容待适配的所有组件，如果无法做到这一点，那么后续一切的适配都将无从谈起。在电脑连接显示器的小例子中，转接头一端可以连接电脑，另一端可以连接显示器。而在上文介绍的类适配器模式的示例程序中，PrintBanner继承了Banner类，同时实现了Print接口。不过细心的朋友们想必已经注意到了，这种模式的局限性是比较大的，如果Print不是一个接口，而是一个类的话，该怎么办呢？Java是单继承的语言，我们是无法同时继承Banner类与Print类的。

这就可以使用本小节欲介绍的对象适配器模式了。照例先给出示例程序的类图：

![2.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/2.jpg)

较之前例，Banner类没有发生任何变化。不同的是，Print由接口变成了一个类。当然，它是一个抽象类，里面的方法printWeak()与printStrong()也均为抽象方法，没有具体的实现。

PrintBanner自然也要做出相应的更改。因为Java是单继承的，PrintBanner无法同时继承Print与Banner。权衡之下，PrintBanner最终还是选择继承了Print，因为从业务层面来说，PrintBanner还是与Print更为接近。对于使用者而言，虽然他实际用的是PrintBanner，但他一直都是当做Print在用的。而Banner更像是为PrintBanner实现功能提供技术支持，因此PrintBanner与Banner之间的关系变为了聚合：Banner成为了PrintBanner的一个成员变量。这样PrintBanner就能使用Banner提供的功能了。

说得简单些，就是Print要求Banner提供功能。Banner一看，发现虽然要的东西自己都有，但是格式却无法完全对上，而它又不想为了这一个单独的功能修改自身。因此它就委托了第三方PrintBanner，并把自己能提供的东西都交给它。PrintBanner并不会添加新的业务逻辑，它只会对Banner提供的东西做出组合和格式上的修改，然后把符合Print要求的内容交给Print。

当然这个委托关系也可以反着理解。Print要求PrintBanner提供功能。PrintBanner本身并没有实现功能，本质上它就相当于一个中间商。它会把需求委托给第三方Banner。当然，Banner并不是专为Print存在的，虽然Print要的东西Banner都有，基本需求也能满足，但是格式并不能完全对上。而PrintBanner则负责这个转换对接的工作。

两种对委托的理解方式本质上是一样的，只是视角不同而已。

本程序中的所有代码将被统一置于design2_2包下，结构如下：

![3.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/3.jpg)

下面来看具体的代码。

**Banner类**

```
package design2_2;

public class Banner {

    private String str;

    public Banner(String str) {
        this.str = str;
    }

    public void showWithParen() {
        System.out.println("(" + this.str + ")");
    }

    public void showWithAster() {
        System.out.println("*" + this.str + "*");
    }
}
```

Banner类较之前例没有变化，在此再次贴出。

**Print类**

```
package design2_2;

public abstract class Print {

    public abstract void printWeak();

    public abstract void printStrong();
}
```

**PrintBanner类**

```
package design2_2;

public class PrintBanner extends Print {

    private Banner banner;

    public PrintBanner(String str) {
        this.banner = new Banner(str);
    }

    @Override
    public void printWeak() {
        this.banner.showWithParen();
    }

    @Override
    public void printStrong() {
        this.banner.showWithAster();
    }
}
```

**Main类**

```
package design2_2;

public class Main {

    public static void main(String[] args) {
        Print print = new PrintBanner("ReimuWang");
        print.printStrong();
        print.printWeak();
    }
}
```

Main类较之前例没有变化，依然原样贴出。

运行Main.java后输出：

```
*ReimuWang*
(ReimuWang)
```

# 登场角色

Adapter模式中有以下角色登场：

**Target(对象)**

对请求者可见的供请求的对象。在"电脑连接显示器"这个例子中，显示器扮演了这个角色。在上文的两段示例程序中，Print 接口/类 扮演了这个角色。

**Client(请求者)**

向Target提出功能申请的请求者。在"电脑连接显示器"这个例子中，观看显示器的人扮演了这个角色。在上文的两段示例程序中，Main类扮演了这个角色。

**Adaptee(被适配)**

提供实际功能的对象。在"电脑连接显示器"这个例子中，电脑扮演了这个角色。在上文的两段示例程序中，Banner类扮演了这个角色。

**Adapter(适配)**

提供Target-Adaptee之间内容格式的转换。在"电脑连接显示器"这个例子中，转接头扮演了这个角色。在上文的两段示例程序中，PrintBanner类扮演了这个角色。

下面给出抽象后，无关语言的类图。

首先是模式一：类适配器模式(使用继承的适配器)：

![4.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/4.jpg)

然后是模式二：对象适配器模式(使用委托的适配器)：

![5.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/5.jpg)

# Adapter模式的应用场景

**针对欲实现的新功能，当前程序已有部分的代码积累**

针对这种情况，除了使用本文介绍的Adapter模式之外，还可以有如下两种做法：

1. 完全无视已有的，类似功能的代码，重新写一套新的。尤其是当已有代码不是自己写的的时候，做出这种选择的程序员的比例会特别的高。因为没人愿意读其他人的代码，绝大多数时候，读他人的代码，进而理解他人的代码，其成本是远高于自己重新写一份的。不过这并不是一种负责的做法，从短期的角度来看，功能的实现是没问题的，开发工作量的减少导致工期得到缩短，上线后出bug的概率也会降低。不过从长远的角度来看，这显然不利于程序的维护。如果负责某个项目的程序员们更倾向于以这种方式管理代码，那么程序中冗余的东西将会越来越多，程序也将越来越难于理解，修改，或增加新功能。直到某一天，程序达到了临界点，不得不进行重构。这仿佛是在击鼓传炸弹，大家都只考虑眼前凑合写着，在谁手上炸算谁倒霉。而且这不仅仅是某几个程序员倒霉的问题，因为这种情况导致的强制重构会耗费大量的人力物力，严重提高程序的维护成本。
2. 修改现有的代码，让它既能满足现在已有的需求，又能满足新的需求。这是我最不推荐的一种方式。因为这意味着这次需求所影响的将不再是它本身了，现在在线上正常运行的功能也会受到影响。那么本次上线需要测试的范围将大幅增加。更糟的是，很多时候，这种影响是无法评估的。很有可能，新功能上线后，一个完全意想不到的地方会报错，好不容易查出来，改好了，另一个地方又错了，进而陷入无休止的解线上bug的噩梦中。

综上，在当前程序已有部分代码积累的情况下，推荐优先使用Adapter模式。

**当前程序并没有新功能相关的代码积累**

针对这种情况，首先我们需要对这次的新功能做出评估，分解抽象出它所需用到的功能点。如果这些功能点很特殊，换句话说，在我们可以预期的未来里，不会再有别的需求可能会复用到这些功能点了，那么就没必要使用Adapter模式，直接写就好了。

反之，如果这些功能点还挺常用的，以后很有可能会有类似的需要这些功能点的需求，那么这些功能点就不要写得太特殊，即不要与本次需求耦合得太紧。我们可以写出一个相对通用的功能点后再使用一个适配器将其特殊化，以完美匹配本次需求。这样，当再有类似需要来了，需要用到这个功能点时，我们就可以再写一个新的适配器，实现代码的复用。

**典型场景：历史版本兼容**

对于各种软件，尤其是客户端软件而言，历史版本的兼容是最基本的需求之一。例如某手机软件，最初客户端的版本是1.0，对应的服务端的版本同样也是1.0。过了一段时间，客户端与服务端的版本均升级为了2.0。服务端是唯一的，握在软件公司手里，因此服务端的升级可以由软件公司自由控制。然而，客户端却是复数个装在用户手机上的，其升级时机完全由用户自身控制。当然，软件公司可以要求用户强制升级：即如果还想用我们的软件，那么必须升级到最新版本才行。不过这有些过于粗暴了，通常只会在跨度特别大的版本更新时才会偶尔使用。如果总是这样做的话，会让用户觉得非常不方便，降低用户对软件的评价。

因此，绝大多数时候，都需要同时向前兼容很多个客户端软件的版本。以上文中版本1.0 --> 版本2.0为例，我们给出示例图。

为了让模型尽可能的简单，我们不妨假设只有1个服务端及两个客户端。最开始大家都是1.0版本时的情况如下图所示：

![6.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/6.jpg)

随后服务端及客户端1都升级为2.0版本，但客户端2并未升级，仍是1.0版本：

![7.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/7.jpg)

这样便起到了兼容历史版本的效果。

# 应用Adapter模式以调用Properties

本程序将统一置于design2_test2包中，结构如下：

![8.jpg](/images/blog_pic/Java 设计模式/2Adapter模式/8.jpg)

Java API中的java.util.Properties类为我们提供了管理键值对的方法。作为一个供所有Java程序员使用的基础组件，它自然不可能照顾到程序员所有的需求。因此我们可以让它扮演Adapter模式中的Adaptee角色。

然后我们可以设计一个FileIO接口以扮演Target的角色：

```
package design2_test2;

import java.io.IOException;
import java.util.Date;

public interface FileIO {

    void readFromFile(String filename) throws IOException;

    void writeToFile(String filename) throws IOException;

    void setTime();

    Date getTime();
}
```

每一个FileIO接口的实现的实例都负责管理一个Properties类的实例。Properties类中提供了大量的用于管理键值对的方法。不过本次需求中只会用到最基础的那一部分，同时由于需求的特殊性，还需要写一些特殊化的方法。

首先介绍一下该程序正常的使用流程：

1. setTime()方法会向Properties实例中以键值对的方式写入当前时间。Key包括year,month,day,hour,minute,second。

2. writeToFile()方法会将设置好的Properties实例写入文件中。

3. readFromFile()方法会读取文件并将其中的内容写入Properties实例。

4. getTime()方法会得到当前Properties实例中存储的时间。

因为只是一个简单的示例程序，因此就不写非正常流程下的容错机制啦。

既然我们将Target角色设计为了一个接口，那么显然，我们是要使用类适配器模式了(当然啦，这也得要Adaptee角色，也就是Properties类允许继承才行)。

然后就是Adapter模式的重点，也就是Adapter角色的扮演者啦：

```
package design2_test2;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Calendar;
import java.util.Date;
import java.util.Properties;

public class FileProperties extends Properties implements FileIO {

    private static final long serialVersionUID = -2256749719985770822L;

    @Override
    public void readFromFile(String filename) throws IOException {
        super.load(new FileInputStream(filename));
    }

    @Override
    public void writeToFile(String filename) throws IOException {
        super.store(new FileOutputStream(filename), "write by FileProperties");
    }

    @Override
    public void setTime() {
        Calendar calendar = Calendar.getInstance();
        super.setProperty("year", calendar.get(Calendar.YEAR) + "");
        super.setProperty("month", calendar.get(Calendar.MONTH) + "");
        super.setProperty("week", calendar.get(Calendar.WEEK_OF_MONTH) + "");
        super.setProperty("day", calendar.get(Calendar.DAY_OF_MONTH) + "");
        super.setProperty("hour", calendar.get(Calendar.HOUR_OF_DAY) + "");
        super.setProperty("minute", calendar.get(Calendar.MINUTE) + "");
        super.setProperty("second", calendar.get(Calendar.SECOND) + "");
    }

    @Override
    public Date getTime() {
        Calendar calendar = Calendar.getInstance();
        calendar.set(Calendar.YEAR, Integer.parseInt(super.getProperty("year")));
        calendar.set(Calendar.MONTH, Integer.parseInt(super.getProperty("month")));
        calendar.set(Calendar.WEEK_OF_MONTH, Integer.parseInt(super.getProperty("week")));
        calendar.set(Calendar.DAY_OF_MONTH, Integer.parseInt(super.getProperty("day")));
        calendar.set(Calendar.HOUR_OF_DAY, Integer.parseInt(super.getProperty("hour")));
        calendar.set(Calendar.MINUTE, Integer.parseInt(super.getProperty("minute")));
        calendar.set(Calendar.SECOND, Integer.parseInt(super.getProperty("second")));
        return calendar.getTime();
    }
}
```

最后是扮演Client角色的Main.java。首先我们先来写一个写入文件的需求：

```
package design2_test2;

import java.io.File;
import java.io.IOException;

public class Main {

    public static void main(String[] args) throws IOException {
        FileIO fileIO = new FileProperties();
        fileIO.setTime();
        fileIO.writeToFile("D:" + File.separator + "test.txt");
    }
}
```

执行该代码，会在D盘下生成新文件test.txt：

```
#write by FileProperties
#Tue Jun 12 16:32:28 CST 2018
hour=16
day=12
second=28
week=3
year=2018
month=5
minute=32
```

注释方面，除了咱们自己指定的字符串"write by FileProperties"之外，Java API还默认的为我们添加了写入文件时的系统时间。

随后我们再来写一个读取文件的需求，待读取的文件自然就是刚刚生成的那个啦：

```
package design2_test2;

import java.io.File;
import java.io.IOException;

public class Main {

    public static void main(String[] args) throws IOException {
        FileIO fileIO = new FileProperties();
        fileIO.readFromFile("D:" + File.separator + "test.txt");
        System.out.println(fileIO.getTime());
    }
}
```

执行该代码后输出如下：

```
Tue Jun 12 16:32:28 CST 2018
```