---
title: Java 设计模式-11.Composite模式
date: 2018-08-03 16:12:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Composite模式被归入了第5部分[一致性]()。在GoF原书中，Composite模式则被归入了[结构型设计模式]()。简单来说，Composite模式可以被描述为：使容器与内容具有一致性，以便于以同等的，递归的方式操作容器与内容。

<!-- more -->

# 综述

在计算机的文件系统中，有名为"文件夹"的概念(也可称为目录)。相对的，还会有名为"文件"的概念。二者的不同之处在于，文件夹是容器，其内部还可放入文件夹或文件。而文件中则无法再放入其他东西了。

不过对于更上层的结构而言，文件夹和文件其实是一回事：例如我们将文件1及文件夹A放入文件夹B中。那么对于文件夹B而言，文件1与文件夹A其实是一类东西：都是文件夹B自身内部存储的内容。此时文件1与文件夹A均可被称为"目录条目"(directory entry)，可以使用相同的策略递归的进行操作。

扩展来说，基本所有的树状结构都可以采用这种思维方式进行组织，例如在视窗系统中，视窗中可以包含子视窗，也可以直接包含内容。GoF将其抽象为了一种设计模式，并命名为Composite模式(composite有"混合物，复合物"的含义)。

# 示例程序

[8.Abstract Factory模式]()的示例程序中应用了完整的Composite模式，因此我们依然使用该示例程序介绍Composite模式。

具体的示例程序细节请参看[8.Abstract Factory模式]()，在此我们仅再贴一下类图：

![0.jpg](/images/blog_pic/Java 设计模式/11Composite模式/0.jpg)

这张类图看起来挺复杂的，不过其中绝大多数的内容都是用于Abstract Factory模式的。如果要研究Composite模式，只需要关注Item,Link,Tray就可以了。

Link代表链接，而Tray代表托盘。托盘中的content字段是一个列表，其中既可以存放其他托盘，也可以直接存放链接。为了能用一个字段同等对待，我们又定义了更抽象的概念Item，从而将Link及Tray统一起来。如图所示，content中存储的实际是Item，至于具体是Link还是Tray，则可依需求而定。

在示例程序中，Item,Link,Tray均是抽象类。不过这是Abstract Factory模式所致。如果只应用Composite模式的话，Item代表的是一个相对抽象的概念，因此往往还会是抽象类，而Link及Tray通常就是可以直接实例化的非抽象类了。

我们可以将Tray视为容器，Link视为内容。使用Composite模式可以使容器与内容具有一致性，也可以称其为多个和单个的一致性，即将多个对象结合在一起，当做一个对象进行处理。

此外，我还想说说类图中Tray内部的add()方法。add()方法的作用为向content中添加Item，只有Tray才需要它。它的可行的定义方式有如下4种：

**1.定义在Item中，抛出异常**

将add()定义在Item中并抛出异常，Link中不进行重写，Tray中依实际功能需求进行重写。

这样做的好处在于，对于调用者而言，可以采用一种统一方式，即只使用Item的实例就可以调用add()方法。示例程序中的Main.java是这样写的：

```
package design8;

import java.io.IOException;

import design8.factory.Factory;
import design8.factory.Page;
import design8.factory.Tray;

public class Main {

    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException, IOException {
        Factory factory = Factory.getListFactory();
        // 创建页面
        Tray tray = factory.createTray(null);
        Page page = factory.createPage("网页导航", tray);
        // 视频网站
        Tray videoTray = factory.createTray("视频网站");
        tray.add(videoTray);
        Tray twoVideoTray = factory.createTray("二次元");
        Tray threeVideoTray = factory.createTray("三次元");
        videoTray.add(twoVideoTray);
        videoTray.add(threeVideoTray);
        twoVideoTray.add(factory.createLink("哔哩哔哩 (゜-゜)つロ 干杯~-bilibili", "https://www.bilibili.com/"));
        twoVideoTray.add(factory.createLink("AcFun弹幕视频网", "http://www.acfun.cn/"));
        threeVideoTray.add(factory.createLink("腾讯视频", "https://v.qq.com/"));
        threeVideoTray.add(factory.createLink("爱奇艺", "http://vip.iqiyi.com/firstsix-new-pc.html"));
        threeVideoTray.add(factory.createLink("优酷", "http://www.youku.com/"));
        // 搜索引擎
        Tray searchTray = factory.createTray("搜索引擎");
        tray.add(searchTray);
        searchTray.add(factory.createLink("百度", "https://www.baidu.com/"));
        searchTray.add(factory.createLink("谷歌", "http://www.google.cn/"));
        // 打印页面
        page.createFile("D:");
    }
}
```

我们在定义Tray实例的引用时，是不能使用Item的，因为此时Item中并没有add()方法。这样的代码抽象度相对较低。而如果我们将add()方法定义在Item中，就可以完美的解决这个问题，只需要在调用时做好异常处理即可。

事实上，这算是相对最优的解决方案了。在Java API的集合框架中，也运用了这个思想：首先贴出AbstractCollection类中的add()方法：

```
public boolean add(E e) {
    throw new UnsupportedOperationException();
}
```

抛出异常，表示不支持添加。而在它的子类AbstractList中，该方法被重写：

```
public boolean add(E e) {
    add(size(), e);
    return true;
}
```

**2.定义在Item中，并声明为抽象方法**

将add()定义在Item中并声明为抽象方法，所有子类依需求决定如何重写。具体来说Link抛出异常，而Tray则写具体的业务逻辑。

本方案的效果与方案1差不多。从易于理解的角度考虑，本方案的思路甚至还要更漂亮一些。不过较之方案1，本方案有一个强制的约束条件：因为我们要将add()方法声明为抽象方法，那么Item就必须为抽象类。这限制了该方案的使用范围。

**3.定义在Item中，并声明为空方法**

将add()定义在Item中并声明为空方法，Link中不进行重写，Tray中依实际功能需求进行重写。

这是很糟糕的做法，虽然这样做也能在调用add()方法时使用Item引用，但是调用者却无法得到足够的反馈：所有的添加仿佛都成功了，即便是不存在添加功能的Item及Link。事实上，如果真实类型不是Tray的话，再调用add()方法，从业务逻辑的角度来讲就是一种异常情况了，要让调用者能感知到才是健壮的代码。

**4.只定义在Tray中**

这也是实例程序中的做法。如果不在乎前几种方案中提到的引用问题(当然我们也可以进行强制类型转换，不过那就太糟了，尽量不要那么做)，那么这是思路最清晰的做法。

# 登场角色

上面的示例程序介绍了Composite模式的Java实现，下面咱们试着跳出语言层面，抽象出Composite模式中登场的角色。

**Content(内容)**

该角色中无法存入其他角色，在示例程序中，由Link类负责扮演。

**Container(容器)**

该角色中可以存入Component角色(具体来说，可以存入Content角色及Container角色)，在示例程序中，由Tray类负责扮演。

**Component(零件)**

该角色是Content角色及Container角色的父类。它的作用是将Content角色及Container角色统一起来。通常来说，Component只是一个概念，是无法实例化的。在示例程序中，由Item类负责扮演。

下面是抽象后，无关语言的类图。关于add()方法，将采用方案1：

![1.jpg](/images/blog_pic/Java 设计模式/11Composite模式/1.jpg)

# 相关设计模式

**[8.Abstract Factory模式]()**

正如示例程序所体现的，实现Abstract Factory模式的零件时往往会用到Composite模式。

**[12.Decorator模式]()**

二者均保证了不同类间的一致性，从而可以以递归的方式去操作它们。