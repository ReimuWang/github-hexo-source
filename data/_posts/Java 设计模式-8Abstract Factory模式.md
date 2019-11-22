---
title: Java 设计模式-8.Abstract Factory模式
date: 2018-07-09 18:03:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Abstract Factory模式被归入了第3部分[生成实例]()。在GoF原书中，Abstract Factory模式则被归入了[创建型设计模式]()。简单来说，Abstract Factory模式可以被描述为：像在工厂中将各个零件组装成产品那样生成实例。

<!-- more -->

# 综述

Abstract Factory的含义为"抽象工厂"。顾名思义，程序会被划分为两层：

第一层：抽象工厂将抽象零件组装成抽象产品。

第二层：具体工厂将具体零件组装成具体产品。

# 示例程序

下面我们给出一个应用Abstract Factory模式的小例子，该例子会创建一个具有层次结构的输出链接集合的html页面。

首先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/8Abstract Factory模式/0.jpg)

本程序中的所有代码将被统一置于design8包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/8Abstract Factory模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

首先是抽象层，统一位于design8.factory包下：

**Item类**

```
package design8.factory;

import java.util.List;

public abstract class Item {

    protected String caption;

    Item(String caption) {
        this.caption = caption;
    }

    public abstract List<String> makeHtml();
}
```

**Link类**

```
package design8.factory;

/**
 * 抽象零件：链接
 */
public abstract class Link extends Item {

    protected String url;

    protected Link(String caption, String url) {
        super(caption);
        this.url = url;
    }
}
```

**Tray类**

```
package design8.factory;

import java.util.ArrayList;
import java.util.List;

/**
 * 抽象零件：托盘。
 * 其中保存着托盘或者链接
 */
public abstract class Tray extends Item {

    protected List<Item> content = new ArrayList<Item>();

    /**
     * @param caption, 为null则表示该tray是最大的那一个托盘
     */
    protected Tray(String caption) {
        super(caption);
    }

    public void add(Item item) {
        this.content.add(item);
    }
}
```

**Page类**

```
package design8.factory;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.PrintStream;
import java.util.List;

/**
 * 抽象产品：页面
 * 产品也可被视为一种零件，或者反过来说也成立。因为此时已经是一种一对一的关系了(该产品只由一种零件构成)
 * 其中保存着网页基本信息及存储主体内容的托盘
 */
public abstract class Page extends Item {

    protected Tray tray;

    protected Page(String caption, Tray tray) {
        super(caption);
        this.tray = tray;
    }

    public void createFile(String outPutPath) throws ClassNotFoundException, NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException, IOException {
        FileOutputStream fos = null;
        PrintStream ps = null;
        try {
            fos = new FileOutputStream(new File(outPutPath + File.separator + this.caption + ".html"));
            ps = new PrintStream(fos);
            List<String> strList = this.makeHtml();
            for (String str : strList) ps.println(str);
        } finally {
            ps.close();
            fos.close();
        }
    }
}
```

**Factory类**

```
package design8.factory;

import design8.listfactory.ListFactory;

/**
 * 抽象工厂
 */
public abstract class Factory {

    public static Factory getListFactory() {
        return ListFactory.getInstance();
    }

    public abstract Link createLink(String caption, String url);

    public abstract Tray createTray(String caption);

    public abstract Page createPage(String caption, Tray tray);
}
```

然后是实现层，统一位于design8.listfactory包下：

**ListLink类**

```
package design8.listfactory;

import java.util.ArrayList;
import java.util.List;

import design8.factory.Link;

public class ListLink extends Link {

    ListLink(String caption, String url) {
        super(caption, url);
    }

    @Override
    public List<String> makeHtml() {
        List<String> result = new ArrayList<String>();
        result.add("<li><a href=\"" + this.url + "\">" + this.caption + "</a></li>");
        return result;
    }
}
```

**ListTray类**

```
package design8.listfactory;

import java.util.ArrayList;
import java.util.List;

import design8.factory.Item;
import design8.factory.Tray;

public class ListTray extends Tray {

    ListTray(String caption) {
        super(caption);
    }

    @Override
    public List<String> makeHtml() {
        List<String> result = new ArrayList<String>();
        if (null == this.caption) this.addLi(result);
        else {
            result.add("<li>");
            result.add(this.caption);
            this.addLi(result);
            result.add("</li>");
        }
        return result;
    }

    private void addLi(List<String> result) {
        result.add("<ul>");
        for (Item item : this.content) result.addAll(item.makeHtml());
        result.add("</ul>");
    }
}
```

**ListPage类**

```
package design8.listfactory;

import java.util.ArrayList;
import java.util.List;

import design8.factory.Page;
import design8.factory.Tray;

public class ListPage extends Page {

    protected ListPage(String caption, Tray tray) {
        super(caption, tray);
    }

    @Override
    public List<String> makeHtml() {
        List<String> result = new ArrayList<String>();
        result.add("<html>");
        result.add("<head><title>" + this.caption + "</title></head>");
        result.add("<body>");
        result.addAll(this.tray.makeHtml());
        result.add("</body>");
        result.add("</html>");
        return result;
    }
}
```

**ListFactory类**

```
package design8.listfactory;

import design8.factory.Factory;
import design8.factory.Link;
import design8.factory.Page;
import design8.factory.Tray;

public class ListFactory extends Factory {

    private static ListFactory SINGLETON = new ListFactory();

    private ListFactory() {}

    public static ListFactory getInstance() {
        return ListFactory.SINGLETON;
    }

    @Override
    public Link createLink(String caption, String url) {
        return new ListLink(caption, url);
    }

    @Override
    public Tray createTray(String caption) {
        return new ListTray(caption);
    }

    @Override
    public Page createPage(String caption, Tray tray) {
        return new ListPage(caption, tray);
    }
}
```

**Main类**

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

执行后，会在D盘根目录下生成网页导航.html：

```
<html>
<head><title>网页导航</title></head>
<body>
<ul>
<li>
视频网站
<ul>
<li>
二次元
<ul>
<li><a href="https://www.bilibili.com/">哔哩哔哩 (゜-゜)つロ 干杯~-bilibili</a></li>
<li><a href="http://www.acfun.cn/">AcFun弹幕视频网</a></li>
</ul>
</li>
<li>
三次元
<ul>
<li><a href="https://v.qq.com/">腾讯视频</a></li>
<li><a href="http://vip.iqiyi.com/firstsix-new-pc.html">爱奇艺</a></li>
<li><a href="http://www.youku.com/">优酷</a></li>
</ul>
</li>
</ul>
</li>
<li>
搜索引擎
<ul>
<li><a href="https://www.baidu.com/">百度</a></li>
<li><a href="http://www.google.cn/">谷歌</a></li>
</ul>
</li>
</ul>
</body>
</html>

```

没进行缩进处理~就这样吧。使用浏览器打开的效果为：

![2.jpg](/images/blog_pic/Java 设计模式/8Abstract Factory模式/2.jpg)

# 登场角色

上面的示例程序介绍了Abstract Factory模式的Java实现，下面咱们试着跳出语言层面，抽象出Abstract Factory模式中登场的角色。

Abstract Factory模式将程序分为了两层。首先是抽象层：

**AbstractComponent(抽象零件)**

之所以使用AbstractComponent而非AbstractProduct，是因为Abstract Factory模式的核心思想之一就是"将复杂的产品拆分为复数个零件，然后再在需要时将零件组装为产品"。当然，我们也可将AbstractProduct视为AbstractComponent的特例：如果某种产品只分解为了1个零件，那么此时零件与产品便是等价的了。

在例子中。抽象零件首先被分为两层。其次第二层又同时包含了身为零件的link，tray及由这两个零件及其他要素组成的最终产品page。本质上其实它们都是同一个角色。

**AbstractFactory(抽象工厂)**

AbstractFactory用于生产AbstractComponent。在示例程序中，由Factory类扮演这个角色。

**ConcreteComponent(具体零件)**

继承AbstractComponent。在例子中，由ListLink类，ListTray类及ListPage类共同扮演这个角色。

**ConcreteFactory(具体工厂)**

ConcreteFactory继承AbstractFactory并生产ConcreteComponent。在例子中，由ListFactory类扮演这个角色。

下面是抽象后，无关语言的类图：

![3.jpg](/images/blog_pic/Java 设计模式/8Abstract Factory模式/3.jpg)

如此的组织结构导致Abstract Factory模式具有如下特点：

- 易于扩展具体的工厂：遵照AbstractFactory中约束的继承规范，再结合本次的需求，扩展出一个新的工厂是很容易的事情。只需要增加新代码，无需对原有代码做任何改变。遵守[开闭原则]()。

- 难以增加新的零件：增加一个新的零件，需要自AbstractFactory起，向下直至所有已有的ConcreteFactory都需要修改。违背了开闭原则。

# 相关设计模式

**[3.Template Method模式]()**
**[4.Factory Method模式]()**
**[7.Builder模式]()**

面试时，我们经常会遇到这样的问题：Factory Method模式与Abstract Factory模式有何异同点？

之所以这样问，是因为二者本就很相近，再加上应用场景的重叠，很多时候我们往往既会使用Factory Method模式，又会使用Abstract Factory模式。再加上二者名称又很相似，引发这样的问题其实也是可以理解的。

在解答这个问题之前，首先需要强调的是，将不同设计模式间的界限分得那么清楚的意义其实不大，这个问题颇有些"茴香豆的茴字有几种写法"的意思。

不过如果我们就是要较真的话，那么其实不应该仅仅局限在这两种设计模式中，而是应该统合以下四种设计模式一起分析：

- Template Method模式
- Factory Method模式
- Builder模式
- Abstract Factory模式

首先，这4种设计模式最终的目的其实是一样的，就是通过某种策略，将一个复杂的问题变得条理清晰并易于扩展。

Template Method模式试图从时间的角度入手解决这个问题：父类中定义方法执行的流程(即从时间上将复杂事件分为很多小段)，子类中负责具体的实现(即具体实现推迟至子类中)。例如父类指定完成一个复杂事件需要经过如下3个步骤：

1. m1()
2. m2()
3. m3()

父类并不会真正的实现这3个方法，具体的实现交给子类。然后通过不同子类间的多态使得代码产出不同的结果。不过，虽然所有的细节(即m1,m2,m3的实现)均是子类完成的。但是子类却没有整个事件的主导权。所有子类无论如何实现m1,m2,m3，它们执行这3个方法的顺序是不能变的。

Factory Method模式则是Template Method模式的特例，它将父类执行的流程限定为了"生成实例"这件事。本质上与Template Method模式是相同的。

而Builder模式则采用了另一种视角，它试图从空间上将一个复杂的产品拆分为各种细小的零件。Builder模式有如下两个角色：设计者及建造者。例如我们要制造一个复杂物体A，它由1个零件B及两个零件C组装而成。那么最基本的，建造者会提供如下方法：

- 生产一个零件B
- 生产一个零件C
- 将传入的零件按要求组装起来

建造者拥有构建复杂物体A所有技术细节的能力。不过单单有建造者是无法构建A的，因为它只能生产零件，却没有设计产品的能力。我们说A是由"1个零件B及两个零件C组装而成"，为什么是这样的零件组合以及如何组装，这些全局上的东西都是设计者的工作。

此外，Template Method模式采用的是ISA的架构，父类与子类间是有层级关系的。而Builder模式采用的则是HASA的架构。设计者与建造者是平级的，我们不能说建造者是设计者的一种，反之亦然。设计者只是能指挥建造者去工作。

虽然Template Method模式与Builder模式采取的策略不同，不过二者之间却有一个共同点：那就是主视角是生产产品，或推动事件的人。在这两个模式的内部，均有负责统筹规划整体思路的角色。对于Template Method模式而言是父类，对于Builder模式而言是设计者。之所以要强调这一点，是为了体现Abstract Factory模式的不同之处。

而Abstract Factory模式，则是基于Template Method模式及Builder模式，结合工厂生产零件这类需求，诞生出的一种更高层级的设计模式。基本上，它是以Builder模式为基础的：工厂生产零件，然后将零件组装成产品。同时它又有着Template Method模式的影子：工厂生产零件与产品的方法被自父类抽象工厂推迟到了子类具体工厂。

不过我认为Abstract Factory模式与Template Method模式及Builder模式有着本质的不同，也就是视角的不同。Template Method模式及Builder模式的主视角是"工人"，并且正如前文已经提到的，不仅视角是"工人"，还会有统筹全局的"大将型"工人，这样在模式内部就可以完成一个完整的需求。而Abstract Factory模式的视角是工厂，工厂是死的，是没有主观能动性的，模式内部也不存在一个统筹全局的工人。正如前文的例子展示的那样，类似统筹全局的工作是在模式外部，也就是main方法中进行的。换句话说，Abstract Factory模式帮助我们构建出了一套完整的，高扩展性的厂房，厂房中虽然配备了工人，却没有设置统筹全局的领导。

---

**[5.Singleton模式]()**

正如前文的例子中展示的，Abstract Factory模式在创建具体工厂时可能会用到Singleton模式。

---

**[11.Composite模式]()**

正如示例程序所体现的，实现Abstract Factory模式的零件时往往会用到Composite模式。

---

**[13.Visitor模式]()**

二者均既遵循又违背了开闭原则。

Abstract Factory模式易于扩展具体的工厂，此时遵循开闭原则。难以增加新的零件，此时违背开闭原则。

Visitor模式易于增加新的Visitor，此时遵循开闭原则。难以修改Structure，此时违背开闭原则。