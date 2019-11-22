---
title: Java 基础-技术体系
date: 2017-10-16 14:30:49
tags: [Java]
categories: Java 基础
---

# JVM的两个无关性

大家较为熟知的是平台无关性。即JVM屏蔽了下层操作系统的技术细节，正如其广告语所言：Write Once,Run Anywhere。

另一个无关性是语言无关性。即实际编写代码的语言并非必须是Java，只要最终给出的是符合规范的平台无关的字节码(ByteCode)文件(Java中特指class文件)，JVM就能正确运行。正因为如此，描述class文件的字节码指令所能表示的语义肯定比Java语法要强，因为其不仅仅可以描述Java语法，还需支持其他语言的语义。详见[JVM-类文件结构](/2017/11/07/JVM-类文件结构/)。语言无关性并没有平台无关性那么普及，主要原因在于其推广很不理想：JVM的设计者是很具野心及攻击性的，其希望将JVM做成一个通用的平台，所有语言都可在其上运行。因此打从一开始，Java虚拟机规范(Java Virtual Machine Specification)与Java语言规范(Java Language Specification)就是分开发布的。从纯技术的角度而言，理论上这是可行的。但目前已成熟的主流语言(C，C++，Python等)，即便有可在JVM上运行的版本，让它们放弃已经非常成熟的自有体系转而投入这个需要做二次转换的平台是很不现实的。

当然，语言无关性也并非一无所成。一些新兴的小众语言(Clojure，JRuby，Groovy，Scala等)选择依托Java这个成熟而又完整的体系来帮助自己节约成本，迅速起飞。从广义上讲，这些语言都是Java技术体系中的一员。

举个小例子：JVM就好比微信公众平台。对于订餐软件而言，理论上都可以使用该平台进行推广。但是对于美团，饿了吗之类的巨头，其投入重点自然是自家的APP而非微信公众号。相反，对于一些小的创业性质的订餐软件而言，独立开发一款APP需要付出高昂的成本，此时利用微信平台所积累的技术及用户基础全力推广公众号将是更好的选择。

虽然一直受挫，但JVM的语言无关性寄托了一个非常美好的愿景：在一个平台上可运行多种语言，意味着同一个程序的不同模块可分别使用不同语言完成，每个语言都做它最擅长的那一部分。同时因为各语言身处同一平台之上，相互之间的沟通也将毫无障碍：跨语言的协作将像调用自身语言原生的API那样自然。

通常我们所说的Java技术体系是狭义的(本文也是如此)：即编写代码的语言特指Java。毕竟这是JVM最为广泛的应用。

<!-- more -->

# Java技术体系

**依功能划分**

即站在软件开发人员的角度思考问题，分为：

- JVM。

- class文件(字节码文件)格式。

- Java语言语法。

- Java API类库。又可细分为核心类库及扩展类库。核心类库一般以java.\*作为包名，主要面向Java SE。扩展类库一般以javax.\*作为包名，主要是针对Java EE对核心类库所作出的补充。但由于历史原因，一部分曾经的扩展类库进入到了核心类库，因此核心类库中也会包含部分javax.\*的代码。

- 第三方Java类库。例如Spring，Mybatis及Maven仓库中的众多框架。

JVM+Java语言语法+Java API类库中的核心类库统称为JDK(Java Development Kit)。JDK是官方所定义的开发一个标准Java应用的最小集，也是Java技术体系中最为重要的一部分，因此很多时候我们都会用JDK来代指Java技术体系。

相对应的，JVM+Java API类库中的核心类库统称为JRE(Java Runtime Environment)，JRE是官方所定义的运行一个标准Java应用的最小集。较之JDK之所以少了Java语言语法，是因为JVM的语言无关性：如果仅仅只为运行程序的话，JVM只需拿到class格式的文件即可。

**依应用平台划分**

即站在用户的角度思考问题，分为：

- Java Card：支持Java小程序(Applet)运行在小内存设备(如智能卡)上的平台。

- Java ME(Micro Edition)：最初被称为J2ME(Java 2 Platform, Micro Edition)。支持Java程序运行在移动终端上的平台。其Java API类库有所精简，并会加入针对移动终端的特殊支持。Java ME与同样是以Java为基础，面向移动终端的安卓(Android)相比是完全不同的两种平台。Java ME所使用的JVM遵循Java虚拟机规范，而安卓则是完全另起炉灶。虽然Java ME的诞生时间较之安卓要早，但近年来安卓如日中天，Java ME则处于不断衰落的边缘化状态。

- Java SE(Standard Edition)：最初被称为J2SE(Java 2 Platform, Standard Edition)。支持面向桌面级应用(使用JDK开发的标准Java应用)的平台。

- Java EE(Enterprise Edition)：最初被称为J2EE(Java 2 Platform, Enterprise Edition)。支持使用多层架构的企业应用(如ERP[Enterprise Resource Planning,企业管理系统])的平台。可看作是Java SE的升级复杂版。