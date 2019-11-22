---
title: Java 设计模式-17.Observer模式
date: 2018-08-28 17:07:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Observer模式被归入了第8部分[管理状态]()。在GoF原书中，Observer模式则被归入了[行为型设计模式]()。

<!-- more -->

# 综述

Observer的含义是"观察者"，既然有观察者，那么自然就要有"观察对象"。在Observer模式中，观察者可以通过某种手段感知到观察对象某些状态的变化，进而做出相应的决策。

# 示例程序

下面我们来看一个应用了Observer模式的示例程序。该程序的观察对象会定时产生随机数，而观察者在感知到随机数变化后会在控制台打印出数值的变化情况。

首先是类图：

![0.jpg](/images/blog_pic/Java 设计模式/17Observer模式/0.jpg)

本程序中的所有代码将被统一置于design17包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/17Observer模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Model类**

```
package design17;

import java.util.Random;

public class Model implements Runnable {

    int number;

    long sleepTime;

    private View view;

    private int numberRange;

    public Model(long sleepTime, int numberRange) {
        this.sleepTime = sleepTime;
        this.numberRange = numberRange;
    }

    @Override
    public void run() {
        Random random = new Random();
        while (true) {
            this.number = random.nextInt(this.numberRange);
            this.view.needUpdate.set(true);
            try {
                Thread.sleep(this.sleepTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void setView(View view) {
        this.view = view;
    }
}
```

**View类**

```
package design17;

import java.util.concurrent.atomic.AtomicBoolean;

public class View implements Runnable {

    AtomicBoolean needUpdate = new AtomicBoolean();

    private Model model;

    private long sleepTime;

    private int lastNumber;

    public View(Model model) {
        this.model = model;
        this.sleepTime = this.model.sleepTime / 10;
    }

    @Override
    public void run() {
        while (true) {
            if (this.needUpdate.get()) {
                System.out.println(this.lastNumber + " --> " + this.model.number);
                this.lastNumber = this.model.number;
                this.needUpdate.set(false);
            }
            try {
                Thread.sleep(this.sleepTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

**Main类**

```
package design17;

public class Main {

    public static void main(String[] args) {
        Model model = new Model(1000L, 50);
        View view = new View(model);
        model.setView(view);
        new Thread(view).start();
        new Thread(model).start();
    }
}
```

执行本类后，会不停的每隔1秒打印一个[0,49]之间的随机整数。截取部分最初的输出如下：

```
0 --> 30
30 --> 46
46 --> 3
3 --> 23
23 --> 15
15 --> 49
49 --> 40
40 --> 29
29 --> 44
44 --> 39
39 --> 11
```

程序的代码和功能都很简单。不过还是有一些需要说明的点：

**Model与View**

在示例程序中，Model类是观察对象，View是观察者。阅读代码后就不难发现，Observer模式与MVC是如此的相似：观察对象对应MVC中的M(model)，观察者对应MVC中的V(view)。这也是本例中类名的用意。

**声明为线程**

在编写设计模式的示例程序时，核心目的是为了介绍设计模式的基本功能点，因此这些设计模式业务逻辑大多非常简单，很少会用到多线程，一般都是单线程跑到底。

而本例的代码虽然简单，却起了共计3个线程：

- main线程
- model线程
- view线程

之所以这么做，是为了模拟通信。如果我们不使用多线程，其实也可以实现看起来差不多的功能：观察对象发生变化后，通知观察者，观察者做出应对。

但是这样仅仅就只能是看起来差不多了。在真正的并发环境下，观察对象发出讯息后工作就结束了，他可以继续做其他事情，也就是说，此时的消息是异步的，这也是现实中实际上发生的情况。而如果只有一个线程，这就仅仅只是看起来发出了一个消息而已，因为此时的消息逻辑是同步的，观察对象必须要等观察者处理完才能做其他事情。

此外，Java其实提供了一套完整的synchronized-wait()-notify()通讯机制，可以更好的完成通讯的功能。不过它隐藏了一些通讯的细节，所以为了更好的描述消息的传递，本示例通过设置flag实现了一个简单的通讯机制(不能由model直接调用view的更新方法，因为这样的话消息又是同步的了)。

**抽象层级**

示例程序并没有将代码分层，之所以这样做，是因为分层是一个基础性的思路了，并不是Observer模式要讨论的重点。在实际应用中，观察对象与观察者均可以分层：

- 观察者的分层是比较常见的。MVC模式中的V通常都会有多个，在接到相同M发来的消息后做出不同的应对。
- 相对来说，观察对象的分层就比较少见了。不过如果业务需要，我们只要设计好观察对象统一的对外接口及消息，一个观察者用相同的套路观察多个同一类的观察对象也是可以的。
- 最后，当然我们也可以设计出多观察者对多观察对象的代码。不过通常是不会有这样的需求的，就算有也要尽量避免，因为这会导致代码层次过于复杂，使人混乱。

**观察对象与观察者的相互感知**

首先，既然叫做观察者，那么观察者必须要知道自己观察的是谁才行。其次，因为观察对象要将自己的变化通知给观察者，因此观察对象也必须能够感知到观察者。反映到代码中，就是我们在main方法中做的model与view的绑定了。

在本示例中，这种对绑定的需求是很明显的，如果我们用了Java提供的synchronized-wait()-notify()通讯机制，这种需求看似是变弱了，但实际上它们只是被封装起来而已。这其实是很好理解的：双向沟通的前提当然是要互相知道对方是谁才行。

这个逻辑其实是有些怪的。因为按照正常的思路来看，观察者需要知道自身观察的对象是理所当然的。但是观察对象其实是无需知道有谁在观察自身的。说明白点，"我就是我，你爱看不看。我变化了也没有义务要通知你"。从更宏观的角度来讲，这其实是因为底层无需对上层负责，就好比父类无需感知到子类那样。

# 示例程序2

在上一个示例程序最后，我们提到了观察对象通知观察者自身变动逻辑上的问题。因此我们会在示例程序2种修改这个逻辑：观察对象无需在意观察者，观察者需要自行想办法获取观察对象的变动情况。本示例将采取观察者轮询观察对象的方式。这也是MVC模式下GUI程序中View最常采取的一种方式。为了使得示例程序更贴近实际情境(或者说更像一个真正的产品)，本示例将使用Java AWT技术来编写可视化的View。

首先是类图：

![2.jpg](/images/blog_pic/Java 设计模式/17Observer模式/2.jpg)

本程序中的所有代码将被统一置于design17_2包下，结构如下：

![3.jpg](/images/blog_pic/Java 设计模式/17Observer模式/3.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Model类**

```
package design17_2;

import java.util.Random;

public class Model implements Runnable {

    int number;

    long sleepTime;

    private int numberRange;

    public Model(long sleepTime, int numberRange) {
        this.sleepTime = sleepTime;
        this.numberRange = numberRange;
    }

    @Override
    public void run() {
        Random random = new Random();
        while (true) {
            this.number = random.nextInt(this.numberRange);
            try {
                Thread.sleep(this.sleepTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

较之前例，Model类并没有什么太大的变化，只是去掉了与View的关联。即，此时的观察对象已无需感知到观察者。

**View类**

```
package design17_2;

import java.awt.Font;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class View extends Frame {

    private static final long serialVersionUID = 1L;

    private Model model;

    private long sleepTime;

    private int lastNumber;

    private int nowNumber;

    public View (Model model) {
        this.model = model;
        this.sleepTime = this.model.sleepTime / 10;
    }

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(300, 150);
        new Thread(this.new RepaintRunnable()).start();
        super.addWindowListener(
            new WindowAdapter() {
                @Override
                public void windowClosing(WindowEvent e) {
                    System.exit(0);
                }
            }
        );
        super.setVisible(true);
    }

    @Override
    public void paint(Graphics g) {
        g.setFont(new Font(null, Font.BOLD, 30));
        if (this.nowNumber != this.model.number) {
            this.lastNumber = this.nowNumber;
            this.nowNumber = this.model.number;
        }
        g.drawString(this.lastNumber + " --> " + this.nowNumber, 100, 100);
    }

    private class RepaintRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    View.this.repaint();
                    Thread.sleep(View.this.sleepTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    @Override
    public void update(Graphics g) {
        Image bImage = super.createImage(this.getWidth(), this.getHeight());
        Graphics bg = bImage.getGraphics();
        this.paint(bg);
        bg.dispose();
        g.drawImage(bImage, 0, 0, this);
    }
}
```

相较之下，View变得就复杂得多了。不过这里大部分都是Java AWT的代码，与Observer模式无关。虽然可能会不太明显，不过Java AWT起了独立线程RepaintRunnable类的实例每隔一段时间定时刷新面板，我们也正是借此完成了观察者对观察对象主动的轮询。

**Main类**

```
package design17_2;

public class Main {

    public static void main(String[] args) {
        Model model = new Model(1000L, 50);
        new Thread(model).start();
        new View(model).launchFrame();
    }
}
```

执行后输出这样的一个面板：

![4.jpg](/images/blog_pic/Java 设计模式/17Observer模式/4.jpg)

面板上的数字会依逻辑实时变动。

# 登场角色

**Observer(观察者)**

在两个示例程序中，均由View类扮演这个角色。

**Subject(观察对象)**

在两个示例程序中，均由Model类扮演这个角色。

因为Observer模式中的观察者与观察对象间的关系灵活且相对简单，因此就不给出无关语言的通用的类图了。

# 观察者也可以兼任改变者

在本文的两个小例子，以及标准的MVC的定义中，观察者的职责是很明确的：它就仅仅负责观察被观察对象，并不会对被观察对象做出修改。不过有些情境下，观察对象在观测到被观察对象发生变化后，或者是其他条件的变化，可能会需要改变观察对象的某些属性。此时观察者相当于兼任了修改者的工作。

我们并不能说这种做法有错。只不过这会导致程序混乱，并且增大程序出错的可能。例如此时稍有不慎，就可能造成无休止的循环：

观察对象变化 --> 观察者观测到这种变化 --> 观察者修改观察对象，导致观察对象再次变化 --> 观察者观测到这种变化...

当然，我们可以通过代码避免这种循环，比如不会对因自身造成的变化进行处理等。不过这样终究是平添了代码逻辑的复杂性，依然不能算是优雅的代码。究其原因，还是因为角色身份的混乱。如果我们实在要实现类似的功能。也应再创建一个修改者的角色，然后通过观察者与修改者的交互达到目的。而观察者依然只有观察这一个使命，保证角色功能的单一性。

# 观察与通知

依笔者的拙见，Observer模式的标准样式应该是示例程序2那样的。然而，在GoF书官方给出的介绍中，示例程序1才是Observer模式的标准调用方式。示例程序2顶多算是Observer模式的变种。对此我是不太服气的，不过争论这些其实也没什么意义(正如一再提到的，设计模式其实没什么标准可言)，日常应用中其实还是示例程序2那种的多见一些。

针对示例程序1那样实现的Observer模式，我们还给它起了一个别名：Publish-Subscribe(发布-订阅)模式。私以为这个命名才是比较贴切的。从我个人的理解来看，我会将示例程序1认为是Publish-Subscribe模式，而将示例程序2认为是Observer模式。二者是同一个大的思路下很相近的两种设计模式。

# Java API对Observer模式的应用

Java API提供了java.util.Observer接口：

```
package java.util;

public interface Observer {

    void update(Observable o, Object arg);
}
```

很显然这是Observer模式中的观察者。而java.util.Observable则是观察对象，这个类就有些复杂了，我们只给出类定义：

```
public class Observable
```

不幸的是，看来Java API的开发人员与我的意见是相左的，他们使用的也是示例程序1的那种方式(不如说好像只有我一个人的想法不一样。看来关于这件事真的就是私下认为一下得了，对外和人沟通时还是采取示例程序1的思路)。观察者的update()方法除了接收观察对象本身之外，还会接收一个附加信息。

不过说实话，Java API提供的这套代码挺鸡肋的。究其原因，还是因为Observable被声明为了一个类，这样的好处自然是Java API的开发人员可以在其中封装很多逻辑，在要求不高的情况下，直接使用也是可以的。只不过Java是单继承的语言，这意味着如果要使用Observer-Observable，观察对象就不能再继承其他父类，不得不说这个限制还是很大的。