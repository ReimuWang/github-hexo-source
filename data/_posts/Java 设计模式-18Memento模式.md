---
title: Java 设计模式-18.Memento模式
date: 2018-08-30 10:30:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Observer模式被归入了第8部分[管理状态]()。在GoF原书中，Observer模式则被归入了[行为型设计模式]()。

<!-- more -->

# 综述

大多数的文本编辑软件，都会支持撤销操作：我们可以通过该操作将文本恢复为之前的版本。而且这种撤销通常可以进行不止一次，即我们能够通过反复撤销将文本恢复为很久以前的版本。

如果打算使用面向对象的语言实现这个功能的话，该怎么做呢？

首先，既然要恢复，就必须将此前的状态保存下来。那么到底该如何进行这个保存操作呢？比较容易想到的方案就是在哪里变更，就在变更之前进行保存。但是既然要保存对象，就必须能够访问对象内部的数据结构才行，如果需要保存的地方有多处，就会使得访问内部数据结构的逻辑散落在代码的多个位置，不利于程序的维护。在编程领域，这被称为"破坏了封装性"。

因此我们引入了Memento模式。Memento有"遗物，纪念品"的含义。顾名思义，它会将对象某个时间点需保存的属性记录下来，形成快照。然后在需要时再拿来使用。这样对对象状态的封装就被封装进了一个个Memento实例中。当外界需要快照时，就调用对象相关的方法生成Memento实例。而当需要恢复为某个Memento实例的状态时，就将该Memento实例传递给对象，让对象自行恢复到以前的状态(颇有些游戏存档-读档的意思)。这样，对于外部而言，对象内部的数据结构依然是黑盒的。

在这里要明确的是，Memento实例和它所记录的对象实例是不同的。这种不同一方面当然来自时间性上，对象本身是有时间维度的，它可能会随着时间的推进变更内部存储的值，而Memento实例则是记录某个时间点上对象的状态，仿佛就是一张定格的照片，是静态的。另一方面，Memento实例也并非是某个时间点上对对象属性的完全复制(拍照片还有失真呢)，当然我们也可以做到完全复制，不过这通常是没有意义的。我们往往只会将我们关心的，又会随时间变化的属性记录下来，用于恢复。

说到这里，大家有没有想到些什么呢？对啦！就是Java的序列化(java.io.Serializable接口)功能。实现了该接口的类就有能力保存自身在某个时间点上的状态，以供后续的恢复。其基本思路和Memento模式还是很像的。只不过，二者的应用目标还是不同的，Java的序列化主要是为了空间上的便利性：即在A地生成的对象，可以通过序列化存储至磁盘上，然后经由网络等通路传递至B地，再恢复为原对象。这是一个完整的对象打包再解包的过程，因此序列化通常会保存对象的所有核心属性(不管是否随时间变化的都会保存)。

# 示例程序

下面我们来看一个应用了Memento模式的小例子。作为一个demo小程序，我们当然不会实现一个记事本。在此我们用Java AWT实现了一个小窗体。小窗体上只有一个数字，表示得分：

![0.jpg](/images/blog_pic/Java 设计模式/18Memento模式/0.jpg)

该程序会监听键盘事件：

- 上箭头：加1分
- 下箭头：减1分
- 左箭头：撤销一次之前的加/减操作
- 右箭头：显示当前存储的快照
- 其他按键：无

下面就赶快来看代码吧~首先是类图：

![1.jpg](/images/blog_pic/Java 设计模式/18Memento模式/1.jpg)

本程序中的所有代码将被统一置于design18包下，结构如下：

![2.jpg](/images/blog_pic/Java 设计模式/18Memento模式/2.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Subject类**

首先是待保存状态的对象：

```
package design18;

public class Subject {

    private int score;

    private String name;

    public Subject(String name) {
        this.name = name;
    }

    public int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }

    public String getName() {
        return name;
    }

    public Memento createSnapshot() {
        return new Memento(this.score);
    }

    public void recovery(Memento memento) {
        this.score = memento.getScore();
    }
}
```

**Memento类**

```
package design18;

public class Memento {

    private int score;

    Memento(int score) {
        this.score = score;
    }

    int getScore() {
        return score;
    }

    @Override
    public String toString() {
        return "Memento [score=" + score + "]";
    }
}
```

虽然代码比较短，但毫无疑问，从名字上也能看出，Memento类就是Memento模式最重要的一个类了。在本示例中，它记录了Subject对象的状态。关于这个类，有如下几点需要说明：

- 单从本文的代码来看，Memento与Subject似乎毫无关联。但实际上，Memento完全附属于Subject。因此实际情况下我们通常会将Memento写为Subject的内部类，或是非public类。本文将Memento单独提取出一个文件，主要是为了能使类图更清晰，便于理解。
- Subject中有name和score两个属性，但是name是不会随时间变化的，因此Memento中就只记录了score。
- 除了toString()之外，Memento中的所有字段和方法的最高访问权限都只到无修饰符，也就是同包下可访问。说实话这个权限还是给大了，因为我们其实希望只有Subject类能访问到这些字段和方法，这也算是没有将Memento写为Subject的内部类的另一个后遗症吧，使得权限控制变得有些混乱了。

接最后一小点。其实我们可以再扩展一下。在面向对象编程中，我们可以按访问权限将方法分为两类，需要注意的是，下文中说的接口指得其实是广义上的方法API，而非狭义上Java中的接口：

- wide interface(宽接口)：暴露大量内部信息的接口，通常只供比较"亲密"的内部人员调用。
- narrow interface(窄接口)：只保留少量非核心的信息，供外部调用。

这样做可以很好的划定模块间的界限，防止对象的封装性被破坏。显然，对于Memento而言，构造函数以及getScore()均是窄接口，而toString()则是宽接口。

**View类**

```
package design18;

import java.awt.Font;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.util.ArrayList;
import java.util.List;

public class View extends Frame {

    private static final long serialVersionUID = 1L;

    private Subject subject;

    private List<Memento> snapshotList = new ArrayList<Memento>();

    public View(Subject subject) {
        this.subject = subject;
    }

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(200, 150);
        super.setTitle(this.subject.getName());
        new Thread(this.new RepaintRunnable()).start();
        super.addWindowListener(
            new WindowAdapter() {
                @Override
                public void windowClosing(WindowEvent e) {
                    System.exit(0);
                }
            }
        );
        super.addKeyListener(
            new KeyAdapter() {
                @Override
                public void keyReleased(KeyEvent e) {
                    switch(e.getKeyCode()) {
                    case KeyEvent.VK_UP:
                        System.out.println("上箭头被按下，加分。");
                        View.this.snapshotList.add(View.this.subject.createSnapshot());
                        View.this.subject.setScore(View.this.subject.getScore() + 1);
                        break;
                    case KeyEvent.VK_DOWN:
                        System.out.println("下箭头被按下，减分。");
                        View.this.snapshotList.add(View.this.subject.createSnapshot());
                        View.this.subject.setScore(View.this.subject.getScore() - 1);
                        break;
                    case KeyEvent.VK_LEFT:
                        System.out.println("左箭头被按下，撤销上一次操作。");
                        if (View.this.snapshotList.size() > 0) {
                            Memento lastMemento = View.this.snapshotList.get(View.this.snapshotList.size() - 1);
                            View.this.snapshotList.remove(View.this.snapshotList.size() - 1);
                            View.this.subject.recovery(lastMemento);
                        }
                        break;
                    case KeyEvent.VK_RIGHT:
                        System.out.println("右箭头被按下，在控制台中打印当前存储的快照。");
                        System.out.println("============================");
                        for (int i = 0; i < View.this.snapshotList.size(); i++)
                            System.out.println(i + "---" + View.this.snapshotList.get(i));
                        System.out.println("============================");
                        break;
                    default:
                        System.out.println("非法按键，键值=" + e.getKeyCode());
                    }
                }
            }
        );
        super.setVisible(true);
    }

    @Override
    public void paint(Graphics g) {
        g.setFont(new Font(null, Font.BOLD, 30));
        g.drawString("" + View.this.subject.getScore(), 50, 100);
    }

    private class RepaintRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    View.this.repaint();
                    Thread.sleep(40);
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

代码看起来比较长，不过大部分都是Java AWT中模板化的东西。和本文最相关的那部分逻辑被封装在了launchFrame()方法的super.addKeyListener()中。从代码中我们可以看到，View对快照的生成过程是完全黑盒的，它只是调用了Subject的createSnapshot()方法，然后将其存储到自身管理的快照列表snapshotList中。而后在需要显示和恢复时，直接使用即可。不过即便是恢复，View也仅仅是将需要恢复为的那份Memento通过Subject的recovery()方法传递给Subject而已，这个过程依然是黑盒的。

**Main类**

```
package design18;

public class Main {

    public static void main(String[] args) {
        new View(new Subject("博丽灵梦")).launchFrame();
    }
}
```

执行该类后，程序即可按照规定的由键盘控制分数的加减。在此输出部分最初的在控制台中打印的文字：

```
上箭头被按下，加分。
右箭头被按下，在控制台中打印当前存储的快照。
============================
0---Memento [score=0]
============================
上箭头被按下，加分。
上箭头被按下，加分。
上箭头被按下，加分。
上箭头被按下，加分。
下箭头被按下，减分。
右箭头被按下，在控制台中打印当前存储的快照。
============================
0---Memento [score=0]
1---Memento [score=1]
2---Memento [score=2]
3---Memento [score=3]
4---Memento [score=4]
5---Memento [score=5]
============================
左箭头被按下，撤销上一次操作。
左箭头被按下，撤销上一次操作。
右箭头被按下，在控制台中打印当前存储的快照。
============================
0---Memento [score=0]
1---Memento [score=1]
2---Memento [score=2]
3---Memento [score=3]
============================
左箭头被按下，撤销上一次操作。
左箭头被按下，撤销上一次操作。
右箭头被按下，在控制台中打印当前存储的快照。
============================
0---Memento [score=0]
1---Memento [score=1]
============================
```

# 登场角色

上面的示例程序介绍了Observer模式的Java实现，下面咱们试着跳出语言层面，抽象出Observer模式中登场的角色。

**Originator(生成者)**

Originator能够生成某时刻表示自身当前状态的Memento角色。也能根据传入的Memento恢复到此前的状态。在示例程序中，由Subject类扮演这个角色。

**Memento(纪念品)**

在示例程序中，由Memento类扮演这个角色。

**Caretaker(负责人)**

Caretaker相当于外部调用人。在需要时它会要求Originator生成Memento，并存储该Memento。不过Caretaker只能调用Memento提供的窄接口，更直白的说，相当于Originator及Memento对Caretaker而言依然还是黑盒的。在合适的时机，Caretaker会将此前的某个Memento传递给Originator，以让Originator恢复为当时的状态。在示例程序中，由View类扮演这个角色。

下面是抽象后，无关语言的类图：

![3.jpg](/images/blog_pic/Java 设计模式/18Memento模式/3.jpg)

# Memento的过期

在前文的分析中，我们说Java提供的序列化在空间上提供备份，Memento模式在时间上提供备份。这样说其实并不完善。从本质上来说，Memento模式就是保存对象某个时间点的快照，以供以后操作。事实上，Memento模式照下的快照也不总是在内存中的。最常见的就是游戏的存档，当游戏退出时，存档通常是会作为存档文件被保存在磁盘上的，下次启动或是需要的时候再读取某个档(SL大法万岁！)。

将快照保存为静态文件提高了程序的灵活性，却也增加了新的风险：我们要注意快照的过期。即随着程序的升级，如果对象的数据结构发生了较大的变化，新程序中的对象可能会无法读取之前版本的快照，此时就会导致快照"过期"。

# Memento的存储技巧

本文示例程序中每个Memento对象需要存储的信息很少。但是很多时候，比如前文提到的游戏存档，每个Memento可能都会占用大量的内存和磁盘空间。最基本的解决办法自然是使用各种压缩技术。不过在此之上，我们还有什么其他的好办法吗？

我们可以参考一下git管理代码的技术。git可以很方便的将代码恢复为之前的版本，从本质上来说实现的基本功能和Memento对象是类似的。只不过，假设我们在git上提交了一次代码，导致代码库由版本1升至了版本2。此时git并不是将版本1全部拍为快照，而是只记录快照间的变化情况。这样在恢复时只需要反向的追溯这种变化即可。在编程领域，这是少有的用时间换空间的例子。

类似的，虽然普通场景下两个连续的快照间的关联没有那么强，但是如果两次变动间绝大多数的数据都没有变化，那么我们也可以只记录发生变化的值，进而构建变化链，达到类似的节省空间的效果。