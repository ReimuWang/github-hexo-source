---
title: Java 设计模式-5.Singleton模式
date: 2018-06-19 10:49:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Singleton模式被归入了第3部分[生成实例]()。在GoF原书中，Singleton模式则被归入了[创建型设计模式]()。简单来说，Singleton模式可以被描述为：只允许生成一个实例。

<!-- more -->

# 综述

类作为模版，在程序运行时通常会生成许多个实例。例如Java中的java.lang.String类与字符串就是一一对应的关系：即如果我们声明了1000个不同的字符串，那么系统中就会有1000个String类的实例与之对应。

但是，有时我们也会有这样的需求：某个类的实例只会有一个。例如东方Project中的八云紫，她的种族是隙间妖怪，属于这个种族的妖怪只有她一个(厉害了我的紫婆婆)。再比如对应于程序配置文件的类，因为配置文件只有一个(假设程序只有一个配置文件)，那么与之对应的代表配置文件的类的实例自然只应该有一个。

为达成这个目的，其实我们可以什么也不用做。只要具体生成实例的代码具备自律性，即对于只需要一个实例的类，我们只调用一次new即可。

这样做的优点是简单，而缺点则是系统健壮性低：设计系统架构是最开始时打下的基础，而new实例这种操作则会随着时间的推进无休止的添加。也许一开始new实例的代码可以保证唯一性，但是随着时间的流逝，new实例的位置越来越多，犯错误的风险自然会越来越大。

类似的例子还有父类方法的重写。如果父类希望所有它的子类都能重写自己的方法m1()，它其实是可以什么都不用做的：只要子类们都自觉重写即可。但是这并非是强制的，如果未来某一天，某个子类决定不重写了，父类将无能为力。因此，为了保证强制力，父类可能会将自身声明为抽象类，进而可以将m1()声明为抽象方法，此时子类再继承该类时，就必须重写方法m1()了。

在保证某类实例的唯一性这个问题上，我们也可以通过某种手段强制保证这点。由此演化而成的设计模式就是Singleton模式。Singleton的含义是"只含有一个元素的集合"，也就是所谓的单例。

# 示例程序

首先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/5Singleton模式/0.jpg)

本程序中的所有代码将被统一置于design5包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/5Singleton模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Singleton类**

```
package design5;

public class Singleton {

    private static Singleton SINGLETON = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return Singleton.SINGLETON;
    }
}
```

Singleton的构造函数的方法体是空的，显式声明它的唯一目的就是为了将它的访问权限标志为private。这样就将Singleton能够实例化的位置限定在自身类的内部了。

外部是通过静态方法getInstance()来获取Singleton的实例的。虽然没有强制的规定，不过通常我们都会将单例模式中获取实例的方法命名为getInstance()。该方法会返回Singleton的类变量SINGLETON。由于SINGLETON的访问权限是private，这样外部就只能通过getInstance()方法来获取它了，进而实现了单例的功能。

在上例中，SINGLETON在声明时直接就初始化了，也就是说，SINGLETON会在类初始化的时候跟着一同初始化。这样的好处是SINGLETON在一开始就会处于一个随时可用的状态，但坏处则是有的时候这是一种浪费，尤其是在SINGLETON占用空间较大时：直到真正需要SINGLETON之前，这部分空间实际上都是浪费的，因此有的时候我们也会这样写单例：

```
package design5;

public class Singleton {

    private static Singleton SINGLETON;

    private Singleton() {}

    public static Singleton getInstance() {
        if (null == Singleton.SINGLETON)
            Singleton.SINGLETON = new Singleton();
        return Singleton.SINGLETON;
    }
}
```

很显然，这又是程序员们熟悉的老话题了。也就是时间和空间上的权衡。

例子1一开始SINGLETON就是可用的，但是却会有一段SINGLETON无用却占用空间的时期。例子2则是每次在需要SINGLETON时去检查它是否初始化，节省了空间却增加了时间上的开销：每次获取实例都要多判断一次if。

例子1因为会在类初始化时就初始化SINGLETON，因此它先天就是线程安全的。而例子2则是线程不安全的，如果需在并发环境下使用，还需对getInstance()方法进行并发控制，例如如下的做法：

```
package design5;

public class Singleton {

    private static Singleton SINGLETON;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (null == Singleton.SINGLETON)
            Singleton.SINGLETON = new Singleton();
        return Singleton.SINGLETON;
    }
}
```

在单例模式中，例子1被称作"饿汉单例"--调用者很饿，调用者不能等，必须尽快返回结果；而例子2则被称作"懒汉单例"--生产者很懒，能拖就拖，不到万不得已不生产。

再扩展来说，"饿"与"懒"在编程领域其实是一组很常见的概念(例如懒加载等)。其核心基本都不出上文论述的藩篱：也就是到底是用时间换空间，还是用空间换时间的问题。

**Main类**

```
package design5;

public class Main {

    public static void main(String[] args) {
        Singleton s1 = Singleton.getInstance();
        Singleton s2 = Singleton.getInstance();
        System.out.println(s1 == s2);
    }
}
```

输出：

```
true
```

说明我们通过多次调用getInstance()方法得到的实例确实是单例的。

# 登场角色

上面的示例程序介绍了Singleton模式的Java实现，下面咱们试着跳出语言层面，抽象出Singleton模式中登场的角色。

**Singleton**

Singleton模式中只有Singleton这一个角色。它可以保证自身只生成一个实例。在示例程序中，由Singleton类来扮演这个角色。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/5Singleton模式/2.jpg)

# 相关设计模式

Singleton模式算是一个比较底层与基础的设计模式，只要是需要确保仅有一个实例的场合，都可以使用单例模式。例如，在以下模式中，很多角色一般都只需要生成一个实例：

- [4.Factory Method模式]()
- [8.Abstract Factory模式]()
- [7.Builder模式]()
- [15.Facade模式]()