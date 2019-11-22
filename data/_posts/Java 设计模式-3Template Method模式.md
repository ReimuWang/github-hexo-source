---
title: Java 设计模式-3.Template Method模式
date: 2018-06-12 16:45:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Template Method模式被归入了第2部分[交给子类]()。在GoF原书中，Template Method模式则被归入了[行为型设计模式]()。简单来说，Template Method模式可以被描述为：在父类中定义处理框架，在子类中进行具体处理。也就是所谓的"将具体处理交给子类"。

<!-- more -->

# 综述

template这个单词的含义为"模版"，在介绍Template Method模式之前，我们有必要先介绍一下模版。

模版是指带有镂空文字的薄薄的塑料板，只要用笔在模版的镂空处进行临摹，即使是手写也能写出整齐的文字。不过，虽然写出的文字的样式都是相同的，但是具体的感觉则是依赖于具体的笔：使用铅笔的话写出的就是铅笔字，使用钢笔的话写出的就是钢笔字。

本文要介绍的Template Method模式就是带有模版功能的模式。作为模版的方法被定义在父类中，它就像镂空的那张模版，只规定样式，因此是抽象的。而具体的实现，也就是到底是铅笔还是钢笔，则被定义在了子类中。

像这样在父类中定义处理流程的框架，在子类中实现具体处理的模式就是Template Method模式。

按照这个定义来说，Template Method模式与多态极为类似。甚至可以认为就是由多态演化出的设计模式。因为在Java中，多态就是同一个行为具有多个不同表现形式或形态的能力。而要称之为多态，则需要3个必要条件：继承，重写，父类引用指向子类对象。很显然，这与Template Method模式的定义完美契合。

当然，多态只是Template Method模式演化的基础，作为一个模式，Template Method模式较之多态还是要复杂一些的：在Template Method模式中，父类里除了有会被子类继承的抽象方法之外，还会有作为模版功能存在的，描述业务流程的非抽象方法存在。

# 示例程序

本程序会将字符或字符串重复输出5次，下面先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/3Template Method模式/0.jpg)

本程序中的所有代码将被统一置于design3包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/3Template Method模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**AbstractDisplay类**

```
package design3;

public abstract class AbstractDisplay {

    private int times;

    protected AbstractDisplay(int times) {
        this.times = times;
    }

    public final void display() {
        this.open();
        for (int i = 0; i < this.times; i++)
            this.print();
        this.close();
    }

    protected abstract void open();

    protected abstract void print();

    protected abstract void close();
}
```

其中，display()是控制业务流程的模版方法。而open()，print()，close()则是交由子类去实现的具体方法。

需要注意的是，display()被强硬的声明为了final，通常来说，这在Template Method模式中是必须的。这其实就是AbstractDisplay作为父类在告诉子类：继承我可以，但是除了必须实现我未实现的抽象方法之外，还不能修改我制定的业务流程。

open()，print()，close()均被声明为了protected，不过这个就没什么强制性了。只不过通常来说，抽象方法就是用来被子类继承的，让外人看到也没用，使用protected只是一个编程时的好习惯。

**CharDisplay类**

```
package design3;

public class CharDisplay extends AbstractDisplay {

    private char ch;

    public CharDisplay(char ch, int times) {
        super(times);
        this.ch = ch;
    }

    @Override
    protected void open() {
        System.out.print("<<");
    }

    @Override
    protected void print() {
        System.out.print(this.ch);
    }

    @Override
    protected void close() {
        System.out.println(">>");
    }
}
```

**StringDisplay类**

```
package design3;

public class StringDisplay extends AbstractDisplay {

    private String str;

    public StringDisplay(String str, int times) {
        super(times);
        this.str = str;
    }

    @Override
    protected void open() {
        this.printLine();
    }

    @Override
    protected void print() {
        System.out.println("|" + this.str + "|");
    }

    @Override
    protected void close() {
        this.printLine();
    }

    private void printLine() {
        System.out.print("+");
        for (int i = 0; i < this.str.length(); i++)
            System.out.print("-");
        System.out.println("+");
    }
}
```

**Main.java**

```
package design3;

public class Main {

    public static void main(String[] args) {
        int times = 5;
        AbstractDisplay adChar = new CharDisplay('蓬', times);
        adChar.display();
        AbstractDisplay adStr = new StringDisplay("I am the bone of my sword", times);
        adStr.display();
    }
}
```

在使用中，AbstractDisplay类型的引用分别指向了其子类CharDisplay及StringDisplay的实例，这里是[里氏替换原则]()的应用。

执行后输出：

```
<<蓬蓬蓬蓬蓬>>
+-------------------------+
|I am the bone of my sword|
|I am the bone of my sword|
|I am the bone of my sword|
|I am the bone of my sword|
|I am the bone of my sword|
+-------------------------+
```

# 登场角色

上面的示例程序介绍了Template Method模式的Java实现，下面咱们试着跳出语言层面，抽象出Template Method模式中登场的角色。

**AbstractClass(抽象类)**

AbstractClass会定义非抽象的模版方法及需由子类角色ConcreteClass具体去实现的抽象方法。在示例程序中，由AbstractDisplay类负责扮演这个角色。

**ConcreteClass(具体类)**

该角色负责实现AbstractClass角色中定义的抽象方法，这些方法将由AbstractClass角色的模版方法调用。在示例程序中，由CharDisplay类及StringDisplay类联袂扮演这个角色。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/3Template Method模式/2.jpg)

# InputStream对Template Method模式的应用

Java API中的抽象类java.io.InputStream应用了Template Method模式。

InputStream的核心的读取流的方法为：

```
public int read(byte b[], int off, int len) throws IOException {
    if (b == null) {
        throw new NullPointerException();
    } else if (off < 0 || len < 0 || len > b.length - off) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return 0;
    }

    int c = read();
    if (c == -1) {
        return -1;
    }
    b[off] = (byte)c;

    int i = 1;
    try {
        for (; i < len ; i++) {
            c = read();
            if (c == -1) {
                break;
            }
            b[off + i] = (byte)c;
        }
    } catch (IOException ee) {
    }
    return i;
}
```

对于输入流的读取而言，无论是什么流，其实质都是由二进制的字节组成的有序集合：二进制字节仿佛水流一样流过。在上面的方法中我们也能看到，其最核心的读取字节的方法是read()，它每次会从特定位置向后再读一个字节。它在InputStream类中的源码为：

```
public abstract int read() throws IOException;
```

至此事情就很清晰了：首先介绍的那个非抽象read()方法是模版方法，其中规定了流程：一个一个取字节，直到取够数为止。而后面介绍的那个抽象的read()方法就是具体的"一个一个取字节"的方法，将由具体的子类实现。

# 相关设计模式

**[4.Factory Method模式]()**
**[7.Builder模式]()**
**[8.Abstract Factory模式]()**

关于这4个设计模式间的联系与区别，详见[8.Abstract Factory模式]()。

**[10.Strategy模式]()**

Strategy模式与Template Method模式均与Java中多态的思想一脉相承。