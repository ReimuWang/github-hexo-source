---
title: Java 设计模式-15.Facade模式
date: 2018-08-13 16:45:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Facade模式被归入了第7部分[简单化]()。在GoF原书中，Facade模式则被归入了[结构型设计模式]()。

<!-- more -->

# 综述

程序终归是会越来越大的，无论我们所采用的架构多么的整洁清晰，代码之间的关系也会越来越复杂(因为业务逻辑本身就是这么复杂)，这是不可避免的。作为调用者，如果我们要使用某个模块，或者是某几个某块联合提供的某个功能是，我们显然是不希望也不需要了解它们内部的调用逻辑的，通常来说，我们需要的就是提出一个请求：我要这个。然后等待收到结果就好了。

基于这种需求，诞生的设计模式就是Facade模式。Facade这个单词源自法语，它的含义是"建筑物的正面"。因此Facade模式也可被翻译为"门面模式"。顾名思义，就是某系统将对外提供的功能都封装在一个门面中，外部请求只和这个门面交互，而无需关心系统内部的复杂逻辑。显然，这是对最少知识法则(迪米特法则)的应用。

# 示例程序

本文将复用[Abstract Factory模式]()的示例。在Abstract Factory模式的示例中，Main.java的代码是这样的：

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

作为Main.java，它承载的其实是调用者的角色。很显然，此时的调用者要了解大量关于抽象工厂的实现细节：

- 需要通过getListFactory()拿到具体的工厂，如果具体的工厂不止一个，还要了解每个工厂的实现细节，自己做出甄别。
- 需要了解各个零件的含义及它们彼此间的关系。
- 需要自己组装零件，设计产品。

在总结Abstract Factory模式时，我们说Abstract Factory模式仅仅只是提供了一套设备齐全的厂房及完成基本零件的工人，但是并没有提供设计产品的厂长。在上文的示例中，这个设计者的工作显然是让调用者来做了。这就导致调用者必须了解大量的抽象工厂的实现细节。而这往往是调用者不希望的。作为调用者我们的愿望通常是这样的：

- 从最简洁的调用方式来说，我们就只调用一个方法，然后工厂为我们生产出默认的定制化的产品。简单来说，就是把main()中的内容全部封装在门面中，供调用者直接调用。
- 退一步来讲，工厂应在门面中提供几种样板样式，比如最终组装好的产品类型1,2,3。或者是接到定制化需求后定制新的产品。调用者拿到的是最终的产品，它们根本不关心这个产品都用了什么零件，这些零件该以何种方式组装，更不会关心具体是哪个工厂生产的零件。

总结一下，抽象工厂就是拥有复杂类关系及业务逻辑的系统，而Facade模式则为它添加了门面，使得调用者无需了解抽象工厂内部的组织关系，直接拿到自己想要的产品。

# 登场角色

上面的示例程序介绍了Facade模式的Java实现，下面咱们试着跳出语言层面，抽象出Facade模式中登场的角色。

**Facade(门面)**

在示例程序中，由Main.java提取出的门面功能组成该角色。

**复杂的系统**

在示例程序中，由抽象工厂的主体组成该角色。

# 一些说明

可能大家已经注意到了，在登场角色这一小节中，我们并没有给出无关语言的类图。

之所以这样做，是因为Facade模式的类关系过于抽象，是无法通过一张具体的类图来描述的。

简单来说，应用Facade模式需要有两个前提：

- 复杂的系统
- 调用者不希望了解系统内部的实现逻辑

此时我们就可以为系统加上一个对外的门面，进而形成Facade模式。

Facade模式是非常灵活的。针对调用方的不同，一个系统可以形成多个门面。同时当多个系统协作构成更大的系统时，我们可以将这些系统的门面(因为门面就相当于是系统的API，外界在满足功能的前提下是无需了解系统的内部实现的)组合起来，此时这些门面又会形成新的，复杂的代码关系。我们便又可以基于这些门面形成新的门面，来作为这个新的大系统对外的接口。从而使得整个程序形成递归的门面结构，让代码逻辑更为清晰。