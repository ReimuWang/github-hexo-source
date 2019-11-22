---
title: Java 基础-发展历史
date: 2017-10-16 16:31:49
tags: [Java]
categories: Java 基础
---

# 1991.4 ----- Java前身Oak

Sun公司(Sun Microsystems，Sun的由来为斯坦福大学校园网[Stanford University Network])的James Gosling领导的绿色计划(Green Project)开发出了一种面向消费性电子产品(如机顶盒，冰箱，收音机)的语言Oak(橡树)，反响平平。

# 1995.5.23 ----- Java语言1.0

随着互联网大潮的袭来，Oak迅速找到了自身的市场定位，并在Sun World大会上改名为Java并发布了1.0版本。同时Java也提出了它那句最为著名的广告语：Write Once,Run Anywhere。

<!-- more -->

# 1996.1.23 ----- JDK1.0

JDK(Java Development Kit)是JVM+Java语言语法+Java API类库中的核心类库的统称。至此Java终于拥有了完整的官方运行环境。

JDK1.0所使用的JVM为Sun Classic VM，这是一个纯解释执行的虚拟机，因此执行代码较慢，给很多人留下了Java运行慢的第一印象并直至今日。实际上，随着JIT及各种优化策略的引入，当今Java在执行速度上已不弱于其他主流语言。

JDK1.0的代表技术包括：

- Applet：包含在Html页面中的Java小程序。

- AWT(Abstract Window Toolkit，抽象窗口工具集)：Java的图形用户界面(Graphical User Interface,GUI)的根基。

# 1996.3

微软申请并获得了Java许可证。

# 1996.4

10个最主要的操作系统供应商申明将在其产品中嵌入Java技术。

# 1996.5月底

Sun公司于美国旧金山举行了首届JavaOne大会，自此其成为了Java使用者每年一度的技术盛会。

# 1996.9

已有大约8.3万个网页应用了Java技术。

# 1997.2.19 ----- JDK1.1

JDK1.1发布了大量Java中最为基础的技术支撑点，例如：

- JDBC(Java DataBase Connectivity)。

- JAR(Java Archive)：java归档文件格式。

- JavaBean。

- RMI(Remote Method Invoke，远程方法调用)。

- 内部类(Inner Class)。

- 反射(Reflection)。

# 1997.9.12 ----- JDK1.1.4

截至1997年9月12日为止，共发布了3个JDK1.1的升级版本：JDK1.1.1~JDK1.1.3。

1997年9月12日JDK1.1.4发布，自此开始基本每个版本都会有一个工程代号，JDK1.1.4的工程代号为Sparkler(宝石)。

# 1997.10

Sun状告微软于1997年发布的Visual J++对Java做出了"不恰当的修改"，这些修改将Java绑定在了Windows平台下，违背了JVM平台无关性的基本规划。

# 1997.12.03 ----- JDK1.1.5

工程代号Pumpkin(南瓜)。

# 1998.4.24 ----- JDK1.1.6

工程代号Abigail(阿比盖尔，女性名)。

# 1998.9.28 ----- JDK1.1.7

工程代号Brutus(布鲁图，古罗马政治家，将军)。

# 1998.12.4 ----- JDK1.2

工程代号Playground(竞技场)。

JDK1.2将Java的技术体系依应用平台划分为3个方向：J2ME(Java 2 Platform, Micro Edition)，J2SE(Java 2 Platform, Standard Edition)，J2EE(Java 2 Platform, Enterprise Edition)。详见[Java 基础-技术体系](/2017/10/16/Java 基础-技术体系/)。

JDK1.2的代表性技术包括：

- EJB(Enterprise JavaBean)：J2EE平台下对JavaBean技术的扩展。

- Java Plug-in：因官司纠纷的原因IE不支持JAVA2的特征，所以Sun设计了该插件以解决这个问题。

- Java IDL(Interface Definition Language)：实现网络上不同平台不同语言之间的交互。

- Swing：新一代GUI，AWT的升级版。

- strictfp(strict float point,精确浮点)关键字。

- Collection框架。

JDK1.2在整个周期中曾存在过3个JVM：

- Sun Classic VM：JDK1.0~JDK1.1中的唯一JVM，JDK1.2中依然以其为默认JVM。

- HotSpot VM。

- Exact VM：只在Solaris平台上出现过。因使用准确式内存管理(Exact Memory Management)而得名(详见[JVM-垃圾收集](/2017/10/26/JVM-垃圾收集/))。

JDK1.2首次引入了JIT(Just In Time)编译器。Sun Classic VM只能以外挂的形式使用JIT编译器，且编译器与解释器无法混合使用，即要么纯编译，要么纯解释(因此执行效率依然糟糕)。HotSpot VM及Exact VM则内置了JIT编译器，且编译器与解释器可以混合执行。

# 1999.3.30 ----- JDK1.2.1

无工程代号。

# 1999.4.8 ----- JDK1.1.8

工程代号Chelsea(切尔西，城市名)。

# 1999.4.27 ----- HotSpot VM发布

HotSpot VM最初由一家名为Longview Technologies的小公司所开发，后该公司于1997年被Sun收购。

与Sun自行研发的Exact VM相比，HotSpot VM具备作为Exact VM名称由来的准确式内存管理技术。相对的，Exact VM同样具备作为HotSpot VM名称由来的热点探测技术。因此二者在核心竞争力上其实相去不远。关于二者究竟哪一个会作为后续版本的主力JVM在Sun内部还曾有过激烈的争论。因此HotSpot VM最终的胜利并非是依托于压倒性的技术优势，而是带有一些"总得选一个吧，那就它吧"的小幸运。

所谓热点探测技术，就是指可以通过执行计数器找到最具编译价值的代码，然后通过JIT编译器以方法为单位进行编译。若方法被频繁调用将会触发标准编译。若方法中的有效循环次数很多，将会触发栈上替换(OSR)。

# 1999.7.8 JDK1.2.2

工程代号Cricket(板球)。

# 2000.5.8 JDK1.3.0

工程代号Kestrel(美洲红隼)。自JDK1.3.0起，HotSpot VM成为默认JVM，Classic VM则作为可选JVM提供。

本次大版本的更新主要集中在类库上，例如数学运算和新的Timer API等。

JNDI(Java Naming and Directory Interface,Java命名和目录接口)开始被作为一项平台级服务被提供(此前仅仅是一项扩展)。

使用CORBA IIOP(Internet Inter-ORB Protocol,互联网内部对象请求代理协议)实现RMI的通信协议。

提供了大量新的Java 2D API(然并卵)，并新增JavaSound类库。

自本版本起，Sun维持了一个习惯：大约每隔两年发布一个大版本，以动物命名工程代号；期间的小版本则以昆虫命名工程代号。

# 2001.5.17 JDK1.3.1

工程代号Ladybird(瓢虫)。

# 2001.1

Sun与微软达成和解。微软承诺会逐步在其产品中移除Java技术。

# 2001.7

微软公布新版的Windows XP将不再支持Sun的JVM，并且推出了.NET平台与Java分庭抗礼。

# 2002.2.13 JDK1.4.0

工程代号Merlin(灰背隼)。在这个大版本的更新中，JDK真正的摆脱了成长期走入了成熟。Compaq(康柏电脑)，Fujitsu(富士通)，SAS(STATISTICAL ANALYSIS SYSTEM)，Symbian(塞班)，IBM(International Business Machines Corporation,国际商业机器公司)等都参与甚至实现了自己独立的JDK1.4。正因为如此，JDK1.4.0至今仍能基本被主流应用(Spring等)所兼容。

JDK1.4带来了很多新特性，例如正则表达式，异常链，NIO(java non-blocking IO或New IO，用以替代此前的阻塞式IO)，日志系统，XML解析器，XSLT(XSL指EXtensible Stylesheet Language,扩展样式表语言，其为XML的扩展。T指转换)转换器等。

JVM方面，彻底移除了Classic VM。

# 2002.9.16 JDK1.4.1

工程代号Grasshopper(蚱蜢)。

# 2003.6.26 JDK1.4.2

工程代号Mantis(螳螂)。

# 2004.9.29 JDK1.5.0

工程代号Tiger(老虎)。自本版本起，官方在宣发上已不再采取类似于JDK1.5.0的命名方式，取而代之的是采用JDK5。只有在程序内部使用的开发版本号(Developer Version，例如java -version的输出)才继续沿用此前的命名方式。为了行文统一，本文在后续描述中将沿用此前的命名方式。

JDK1.5做出了自JDK1.2以来在语法层面上最大幅度的改进。引入了自动装箱，泛型，动态注解，枚举，可变长参数，foreach等新功能点。

JVM层面改进了Java的内存模型JMM(Java Memory Model)。

API层面引入了java.util.concurrent并发包实现了一个粗粒度的并发框架。

# 2006.11.13 开源

在当日的JavaOne大会上，Sun宣布将对Java进行开源。并建立起OpenJDK组织管理这些开源代码。在随后的一年多的时间内，除了极少量的私有的与语言特性基本无关的代码以外，OpenJDK基本可与官方的JDK等同。

OpenJDK中所用的JVM与官方的JDK相同，即也为HotSpot VM。

# 2006.12.11 JDK1.6.0

工程代号Mustang(野马)。自本版本起sun终结了J2SE,J2ME,J2EE，转而使用Java SE 6,Java ME 6,Java EE 6。

该版本新增的功能包括：通过内置Mozilla JavaScript Rhino引擎的方式提供动态语言支持。提供编译APT。提供微型HTTP服务器API。

JVM方面，改进了锁与同步，垃圾收集，类加载等方面的算法。

# 2009.4.20 Oracle收购Sun

此前Oracle已收购了BEA并从其手中接收了三大商用JVM之一的JRockit(主要面向的市场为服务器端，因此可以不必考虑启动时间及响应时间，JRockit内部仅有即时编译器而没有解释器)。收购Sun后Oracle又从其手中接收了三大商用JVM中的第二个，也是最为流行的HotSpot。Oracle计划未来将会将这两款JVM合二为一，形成HotRockit。合并后的JVM将兼具两家之长，譬如使用JRockit的垃圾回收器与MissionControl(任务控制，是一种调优工具)。使用HotSpot的JIT编译器等。

三大商用JVM中的最后一个为IBM公司的J9，该虚拟机主要用于支持IBM自身的Java设备。其应用范围和HotSpot基本一致。

# 2011.7.28 JDK1.7.0

工程代号Dolphin(海豚)。

JDK1.7.0立项时共计划了10个里程碑，原始计划至2010年9月9日完成所有里程碑。2009年2月19日完成了第一个里程碑后，因为Sun公司经营上的问题，进度将无法如期完成。

在Sun被Oracle收购后，为保证JDK1.7.0能在2011年7月28日准时发布，只能放弃部分计划中的功能点：例如Lambda项目(Lambda表达式，函数式编程)，Jigsaw项目(虚拟机模块化支持)，Coin项目(语言细节进化)等。

最终，JDK1.7的功能为：动态语言支持，GarbageFirst(G1)收集器(刚推出时G1依然处于Experimental状态，直到2012年4月的Update 4才转为正式版)，加强对非Java语言的调用支持(JSR-292，直至JDK1.7周期结束也未完全完成)，改进类加载架构等。

API层面引入了java.util.concurrent.forkjoin包完善了java.util.concurrent的并发框架。