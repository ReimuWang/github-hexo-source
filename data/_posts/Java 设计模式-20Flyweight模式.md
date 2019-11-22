---
title: Java 设计模式-20.Flyweight模式
date: 2018-09-04 17:41:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Flyweight模式被归入了第9部分[避免浪费]()。在GoF原书中，Flyweight模式则被归入了[结构型设计模式]()。

<!-- more -->

# 综述

flyweight这个单词的释义为"次最轻量级的拳击选手"。顾名思义，Flyweight模式的目的就是让对象"变轻"。更具体的来说，是让对象消耗的资源变少。其中最易于理解的消耗自然就是内存占用了。通俗来说，Flyweight模式就是在通过共享实例来尽量避免new出实例(少进行new的操作其实也相当于减少了时间的消耗，只是这没有内存减耗那么明显)。JVM所管理的"字符串常量池"遵循的其实也是类似的思想。

# 示例程序

下面来看一段应用了Flyweight模式的小例子。首先我们有0-9这10个数字的10张png图片：

![0.jpg](/images/blog_pic/Java 设计模式/20Flyweight模式/0.jpg)

我们希望在输入一段数字后，程序能通过拼接这10张素材图片，从而生成一张新的图片。这里就用我的女神，苏联卫国战争时期的王牌女飞行员，被称为"斯大林格勒的白百合"的莉莉娅的生日为例吧

女神(↓↓)

![1.jpg](/images/blog_pic/Java 设计模式/20Flyweight模式/1.jpg)

输入19210818后，程序最终会输出：

![2.jpg](/images/blog_pic/Java 设计模式/20Flyweight模式/2.jpg)

在具体编写代码之前，我们可以先思考一下如何去做。最为简单的思路就是需要什么我们就去加载什么。如果我们将每一张原始的素材图片都视为一个对象的话，那么，对于19210818而言，毫无疑问，我们需要生成8个这样的图片对象。然后将这8个对象拼接为一个新的对象。显然，这是可行的做法。但这真的是最优的做法吗？

当然不是。

细心的朋友们应该都已经注意到了，虽然最终拼接需要8个对象，但实际用到的数字只有"19208"这5个，有3个数字是重复的。省掉这3个重复的数字的空间就是Flyweight模式要做的工作。

下面就赶快来看代码吧~首先是类图：

![3.jpg](/images/blog_pic/Java 设计模式/20Flyweight模式/3.jpg)

本程序中的所有代码将被统一置于design20包下，结构如下：

![4.jpg](/images/blog_pic/Java 设计模式/20Flyweight模式/4.jpg)

其中Main.java是测试代码，并没有出现在类图中。而img包下则是前文展示的那10张数字素材图片。

下面将逐个贴出每个类的源码。

**Img类**

```
package design20;

import java.awt.image.BufferedImage;
import java.io.IOException;

import javax.imageio.ImageIO;

import design19.view.View;

public class Img {

    int number;

    BufferedImage image;

    Img(int number) {
        try {
            this.image = ImageIO.read(View.class.getClassLoader().getResource("design20/img/" + number + ".png"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**ImgFactory类**

```
package design20;

import java.awt.image.BufferedImage;
import java.util.HashMap;
import java.util.Map;

public class ImgFactory {

    private Map<Integer, Img> pool = new HashMap<Integer, Img>();

    private static ImgFactory SINGLETON;

    public static final int WIDTH = 88;

    public static final int HEIGHT = 151;

    private ImgFactory() {}

    public static ImgFactory getInstance() {
        if (null == ImgFactory.SINGLETON) ImgFactory.SINGLETON = new ImgFactory();
        return ImgFactory.SINGLETON;
    }

    public BufferedImage getNumImg(int number) {
        if (!this.pool.containsKey(number)) this.pool.put(number, new Img(number));
        return this.pool.get(number).image;
    }
}
```

从逻辑的角度分析，生产Img的工厂是一个独立的个体，因此应该允许其生成实例而非使用类方法。同时工厂又只需要有一个，因此应用了[5.Singleton模式]()。

对外界而言，是否应用Flyweight模式并没有什么不同。因为他们只是在调用getNumImg()方法获得自己想要的数字对应的图片。至于这个图片是新生成的还是复用之前已经存在的，他们并不在意。而在ImgFactory内部，我们创建了一个存储Img的"池"pool：调用方传入的数字千千万万，但池中最多只会存储0-9共计十张图片，这就从一定程度上节省了内存的开销。

虽然与Flyweight模式无关，本程序还在试图从另一个维度上节省开销。本文在创建Img的策略上采用了懒加载，初始时pool中是空的，只有当获取某个具体数字对应的图片时才会检查该图片是否已在pool中，如果在则返回，反之则创建后返回。这相当于将图片初始化的时间由pool的初始化推迟至调用时。不仅如此，我们还可以很容易想到，如果某个数字就是不会被调用，那么它永远不会被初始化，和一开始就全部初始化好相比，不仅节省了时间，还节省了空间。

当然，懒加载对时间的节省只是相对的。它能让程序启动时间大大减少。但正所谓出来混总是要还的，既然要使用，那么终究还是要初始化的，懒加载只是将初始化的实际延后了。不仅如此，这种延后还要付出更大的时间代价：如果最初就将pool中的图片全部初始化好，那么在getNumImg()时直接返回即可，因为我们确信此时pool中已经有了所有需要的图片了。反之，使用懒加载后，每次取图片时都需判断图片是否已经生成，反而相当于增加了时间开销。因此，是否使用懒加载，完全是要根据实际情况具体问题具体分析的。

**Main类**

```
package design20;

import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;

public class Main {

    public static void main(String[] args) throws IOException {
        Main.createImg("19210818");
    }

    private static void createImg(String str) throws IOException {
        int length = str.length();
        BufferedImage bufferedImage = new BufferedImage(ImgFactory.WIDTH * length, ImgFactory.HEIGHT, BufferedImage.TYPE_INT_RGB);
        for (int i = 0; i < length; i++) {
            int tempNum = Integer.parseInt(str.substring(i, i + 1));
            BufferedImage tempImg = ImgFactory.getInstance().getNumImg(tempNum);
            for (int y = 0; y < tempImg.getHeight(); y++)
                for (int x = 0; x < tempImg.getWidth(); x++)
                    bufferedImage.setRGB(x + i * ImgFactory.WIDTH, y, tempImg.getRGB(x, y));
        }
        ImageIO.write(bufferedImage, "jpg", new File("D:\\img.jpg"));
    }
}
```

执行后，即可得到前文展示的结果。

# 登场角色

上面的示例程序介绍了Flyweight模式的Java实现，下面咱们试着跳出语言层面，抽象出Flyweight模式中登场的角色。

**Flyweight(轻量级类)**

即可共享的类，在示例程序中，由Img类扮演这个角色。

**FlyweightFactory(轻量级类工厂)**

即控制Flyweight共享情况的工厂。在示例程序中，由ImgFactory类扮演这个角色。

下面是抽象后，无关语言的类图：

![5.jpg](/images/blog_pic/Java 设计模式/20Flyweight模式/5.jpg)

# Intrinsic与Extrinsic

Flyweight模式的核心在于共享实例。这里隐含着一个前提，那就是必须先判断实例能否被共享(因为共享实例的改变，会导致所有用到它的地方所得到的实例均会发生变化)，只有能被共享的实例，才有资格进一步考虑到底要不要共享。在编程领域，可以被共享的信息被称为Intrinsic信息，Intrinsic意思是"本质的，固有的"，顾名思义，这是指那些不会因调用环境不同而改变的信息；相对的，不能被共享的信息被称为Extrinsic信息，Extrinsic的含义是"外在的，非本质的"。

依这种思路来分析，类字段均应是Intrinsic信息，被定义为单例的实例中存储的也均应是Intrinsic信息。

在本示例中，每个数字对应的图片是唯一且不可变的，属于Intrinsic信息，当我们输入数字"11"时，这两个1对应的都是那同一张代表1的图片，因此这些图片实例可以成为Flyweight角色被共享。