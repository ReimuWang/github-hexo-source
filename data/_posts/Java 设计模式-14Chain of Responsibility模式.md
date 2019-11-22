---
title: Java 设计模式-14.Chain of Responsibility模式
date: 2018-08-13 15:45:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Chain of Responsibility模式被归入了第6部分[访问数据结构]()。在GoF原书中，Chain of Responsibility模式则被归入了[行为型设计模式]()。

<!-- more -->

# 综述

假设我们要去某公司领取材料。首先我们去公司的"前台"，"前台"告诉说去问"营业窗口"。到了"应该窗口"后又被告知要去"售后部门"，而后"售后部门"又说应该去"资料中心"，最后我们终于在"资料中心"取到了数据。

在上文的小例子中，我们被像皮球一样在各部门之间踢来踢去。对于取资料的人而言，公司的做法是推卸责任，显然是很不友好的。但是从公司的角度而言，这样做却是有其正面意义的：这样做可以明确各个部分的职责，同时控制对外暴露的模块。在上例中，对外暴露的仅仅就只有"前台"，我们之所以知道下一步要去"营业窗口"，那是"前台"告知的。如果"前台"告知的是别的地方，那么我们就会去别的被告知的地方。后续所有的模块所做的工作都是一样的，直到我们在"资料中心"拿到资料。

在编程领域中，这种处理问题的方式被称为"责任链"。由此发展而来的设计模式即为Chain of Responsibility模式。在这种模式下，责任链上的节点对自身能处理的问题是很明确的，一旦接到责任链上游传递过来的问题后，会首先判断自身能否处理，如果不能处理，则传递给下游。

# 示例程序

下面我们给出一个小例子，首先是类图：

![0.jpg](/images/blog_pic/Java 设计模式/14Chain of Responsibility模式/0.jpg)

本程序中的所有代码将被统一置于design14包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/14Chain of Responsibility模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Trouble类**

```
package design14;

public class Trouble {

    private int number;

    public Trouble(int number) {
        this.number = number;
    }

    public int getNumber() {
        return number;
    }

    @Override
    public String toString() {
        return "[Trouble " + this.number + "]";
    }
}
```

**Handler类**

```
package design14;

public abstract class Handler {

    private String name;

    private Handler next;

    public Handler(String name) {
        this.name = name;
    }

    public Handler setNext(Handler next) {
        this.next = next;
        return this.next;
    }

    public static final Handler init() {
        Handler result = new NoHandler();
        result.setNext(new SpecialHandler(5)).setNext(new LimitHandler(20)).setNext(new OddHandler());
        return result;
    }

    public final void handle(Trouble trouble) {
        if (this.resolve(trouble))
            System.out.println(trouble + "is resolved by" + this);
        else if (null != this.next) {
            System.out.println(this + " can't resolve " + trouble + ",pass to next handler:" + this.next);
            this.next.handle(trouble);
        } else
            System.out.println("all handler can't resolve " + trouble);
    }

    protected abstract boolean resolve(Trouble trouble);

    @Override
    public String toString() {
        return "[Handler " + this.name + "]";
    }
}
```

说明

- setNext()方法返回Handler的目的是可以像链接一样在一个语句内一直set下去。思路类似于StringBuilder的append()方法。
- handle()中应用了[3.Template Method模式]()，resolve()就是需要子类实现的抽象方法。
- 责任链的初始化被作为静态方法放到了Handler中。实际应用时可依需求灵活调整。
- 显然，责任链不是一成不变的，我们可以依需求灵活的调整节点的顺序及功能，从而形成各种各样的责任链。

接下来的4个类是具体的解决问题的子类：

**NoHandler类**

```
package design14;

/**
 * 不处理任何问题
 */
public class NoHandler extends Handler {

    public NoHandler() {
        super("NoHandler");
    }

    @Override
    protected boolean resolve(Trouble trouble) {
        return false;
    }
}
```

**SpecialHandler类**

```
package design14;

public class SpecialHandler extends Handler {

    private int number;

    public SpecialHandler(int number) {
        super("SpecialHandler-" + number);
        this.number = number;
    }

    @Override
    protected boolean resolve(Trouble trouble) {
        if (trouble.getNumber() == this.number) return true;
        return false;
    }

}
```

**LimitHandler类**

```
package design14;

/**
 * 只处理编号小于特定值的问题
 */
public class LimitHandler extends Handler {

    private int limit;

    public LimitHandler(int limit) {
        super("LimitHandler-" + limit);
        this.limit = limit;
    }

    @Override
    protected boolean resolve(Trouble trouble) {
        if (trouble.getNumber() < this.limit) return true;
        return false;
    }
}
```

**OddHandler类**

```
package design14;

/**
 * 只处理编号为奇数的问题
 */
public class OddHandler extends Handler {

    public OddHandler() {
        super("OddHandler");
    }

    @Override
    protected boolean resolve(Trouble trouble) {
        if (trouble.getNumber() % 2 == 1) return true;
        return false;
    }
}
```

**Main类**

```
package design14;

public class Main {

    public static void main(String[] args) {
        Handler handler = Handler.init();
        for (int i = 0; i < 30; i++)
            handler.handle(new Trouble(i));
    }
}
```

输出：

```
[Handler NoHandler] can't resolve [Trouble 0],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 0],pass to next handler:[Handler LimitHandler-20]
[Trouble 0]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 1],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 1],pass to next handler:[Handler LimitHandler-20]
[Trouble 1]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 2],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 2],pass to next handler:[Handler LimitHandler-20]
[Trouble 2]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 3],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 3],pass to next handler:[Handler LimitHandler-20]
[Trouble 3]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 4],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 4],pass to next handler:[Handler LimitHandler-20]
[Trouble 4]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 5],pass to next handler:[Handler SpecialHandler-5]
[Trouble 5]is resolved by[Handler SpecialHandler-5]
[Handler NoHandler] can't resolve [Trouble 6],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 6],pass to next handler:[Handler LimitHandler-20]
[Trouble 6]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 7],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 7],pass to next handler:[Handler LimitHandler-20]
[Trouble 7]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 8],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 8],pass to next handler:[Handler LimitHandler-20]
[Trouble 8]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 9],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 9],pass to next handler:[Handler LimitHandler-20]
[Trouble 9]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 10],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 10],pass to next handler:[Handler LimitHandler-20]
[Trouble 10]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 11],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 11],pass to next handler:[Handler LimitHandler-20]
[Trouble 11]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 12],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 12],pass to next handler:[Handler LimitHandler-20]
[Trouble 12]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 13],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 13],pass to next handler:[Handler LimitHandler-20]
[Trouble 13]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 14],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 14],pass to next handler:[Handler LimitHandler-20]
[Trouble 14]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 15],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 15],pass to next handler:[Handler LimitHandler-20]
[Trouble 15]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 16],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 16],pass to next handler:[Handler LimitHandler-20]
[Trouble 16]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 17],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 17],pass to next handler:[Handler LimitHandler-20]
[Trouble 17]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 18],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 18],pass to next handler:[Handler LimitHandler-20]
[Trouble 18]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 19],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 19],pass to next handler:[Handler LimitHandler-20]
[Trouble 19]is resolved by[Handler LimitHandler-20]
[Handler NoHandler] can't resolve [Trouble 20],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 20],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 20],pass to next handler:[Handler OddHandler]
all handler can't resolve [Trouble 20]
[Handler NoHandler] can't resolve [Trouble 21],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 21],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 21],pass to next handler:[Handler OddHandler]
[Trouble 21]is resolved by[Handler OddHandler]
[Handler NoHandler] can't resolve [Trouble 22],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 22],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 22],pass to next handler:[Handler OddHandler]
all handler can't resolve [Trouble 22]
[Handler NoHandler] can't resolve [Trouble 23],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 23],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 23],pass to next handler:[Handler OddHandler]
[Trouble 23]is resolved by[Handler OddHandler]
[Handler NoHandler] can't resolve [Trouble 24],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 24],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 24],pass to next handler:[Handler OddHandler]
all handler can't resolve [Trouble 24]
[Handler NoHandler] can't resolve [Trouble 25],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 25],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 25],pass to next handler:[Handler OddHandler]
[Trouble 25]is resolved by[Handler OddHandler]
[Handler NoHandler] can't resolve [Trouble 26],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 26],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 26],pass to next handler:[Handler OddHandler]
all handler can't resolve [Trouble 26]
[Handler NoHandler] can't resolve [Trouble 27],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 27],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 27],pass to next handler:[Handler OddHandler]
[Trouble 27]is resolved by[Handler OddHandler]
[Handler NoHandler] can't resolve [Trouble 28],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 28],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 28],pass to next handler:[Handler OddHandler]
all handler can't resolve [Trouble 28]
[Handler NoHandler] can't resolve [Trouble 29],pass to next handler:[Handler SpecialHandler-5]
[Handler SpecialHandler-5] can't resolve [Trouble 29],pass to next handler:[Handler LimitHandler-20]
[Handler LimitHandler-20] can't resolve [Trouble 29],pass to next handler:[Handler OddHandler]
[Trouble 29]is resolved by[Handler OddHandler]
```

# 登场角色

上面的示例程序介绍了Chain of Responsibility模式的Java实现，下面咱们试着跳出语言层面，抽象出Chain of Responsibility模式中登场的角色。

**Handler(处理者)**

抽象父类，在示例程序中，由Handler类扮演这个角色。

**ConcreteHandler(具体的处理者)**

在示例程序中，由NoHandler，SpecialHandler，LimitHandler，OddHandler类联袂扮演这个角色。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/14Chain of Responsibility模式/2.jpg)

从最本源的角度上来讲，Chain of Responsibility模式其实是不需要一个抽象的父类的，也就是不需要Handler：只要ConcreteHandler之间能形成链即可。不过通常来说，穿在同一个链子上的ConcreteHandler所能处理的问题终归是一个类型的，因此会再抽象出一个层级。

使用Chain of Responsibility模式可以弱化调用者及处理者之间的关联，也不需要有一个统筹全局的角色决定某问题究竟该由哪个ConcreteHandler处理，问题会沿着设定好的链子走下去，直到解决或者链子结束(不过话说回来，统筹全局的角色其实还是存在的，决定链子如何拼接其实就是一种全局的工作，只不过这种做法是静态的)。

较之有一个特定的决策者决定问题的处理人，使用Chain of Responsibility模式会增大处理问题的时间陈本，在对效率要求较高的场合应谨慎使用(话是这么说没错啦，不过推卸责任时通常仅仅只是判断一下自身能不能处理，一般是不会花费太多时间的)。