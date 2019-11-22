---
title: Java 设计模式-7.Builder模式
date: 2018-07-05 18:12:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Builder模式被归入了第3部分[生成实例]()。在GoF原书中，Builder模式则被归入了[创建型设计模式]()。简单来说，Builder模式可以被描述为：通过各个阶段的处理以组装出复杂的实例。

<!-- more -->

# 综述

在英语中，build通常用来指建造大型的建筑物。对于这种具有复杂建筑结构的大型物体而言，很难做到一蹴而就。我们往往需要分模块的建造各个部分，然后在合适的时机将它们组装到一起。将这种思想延伸到编程领域，诞生的就是Builder模式。

# 示例程序

下面我们给出一个应用Builder模式的小例子，该例子会根据需求打印出txt或html两种样式的文档。

首先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/7Builder模式/0.jpg)

本程序中的所有代码将被统一置于design7包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/7Builder模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Builder类**

```
package design7;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.PrintStream;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;

public abstract class Builder {

    protected List<String> result = new ArrayList<String>();

    public abstract void bulidHead(String head);

    public abstract void bulidTitle(String title);

    public abstract void bulidString(String str);

    public abstract void bulidItems(String[] items);

    public abstract void bulidTail();

    public void createFile(String pathPre) throws ClassNotFoundException, NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException, IOException {
        Field filed = this.getClass().getDeclaredField("FILE_TYPE");
        filed.setAccessible(true);
        String fileType = (String)filed.get(null);
        String outPutPath = pathPre + "." + fileType;
        FileOutputStream fos = null;
        PrintStream ps = null;
        try {
            fos = new FileOutputStream(new File(outPutPath));
            ps = new PrintStream(fos);
            for (String str : this.result) ps.println(str);
        } finally {
            ps.close();
            fos.close();
        }
    }
}
```

**TextBuilder类**

```
package design7;

public class TextBuilder extends Builder {

    @SuppressWarnings("unused")
    private static String FILE_TYPE = "txt";

    @Override
    public void bulidHead(String head) {
        String decorate = "====================";
        this.result.add(decorate + head + decorate);
    }

    @Override
    public void bulidTitle(String title) {
        String decorate = "#";
        this.result.add(decorate + title + decorate);
    }

    @Override
    public void bulidString(String str) {
        this.result.add(str);
    }

    @Override
    public void bulidItems(String[] items) {
        for (String item : items)
            this.result.add("    · " + item);
    }

    @Override
    public void bulidTail() {}
}
```

**HtmlBuilder类**

```
package design7;

public class HtmlBuilder extends Builder {

    @SuppressWarnings("unused")
    private static String FILE_TYPE = "html";

    @Override
    public void bulidHead(String head) {
        this.result.add("<html>");
        this.result.add(this.indent(1) + "<head><title>" + head + "</title></head>");
        this.result.add(this.indent(1) + "<body>");
    }

    @Override
    public void bulidTitle(String title) {
        this.result.add(this.indent(2) + "<h1>" + title + "</h1>");
    }

    @Override
    public void bulidString(String str) {
        this.result.add(this.indent(2) + "<p>" + str + "</p>");
    }

    @Override
    public void bulidItems(String[] items) {
        this.result.add(this.indent(2) + "<ul>");
        for (String item : items)
            this.result.add(this.indent(3) + "<li>" + item + "</li>");
        this.result.add(this.indent(2) + "</ul>");
    }

    @Override
    public void bulidTail() {
        this.result.add(this.indent(1) + "</body>");
        this.result.add("</html>");
    }

    private String indent(int level) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < level; i++) sb.append("  ");
        return sb.toString();
    }
}
```

**Designer类**

```
package design7;

import java.io.IOException;

public class Designer {

    public void construct(Builder builder, String path, String title) throws ClassNotFoundException, NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException, IOException {
        builder.bulidHead(title);
        builder.bulidTitle("红魔馆");
        builder.bulidString("红魔馆（こうまかん/Koumakan）是蕾米莉亚·斯卡雷特拥有的一间房子。它位于雾之湖的边缘。从外面可以看见深红色的窗户，但是窗户并不多，因为吸血鬼不喜欢阳光。里面也有很多没有窗户的房间。屋顶上有一个钟楼，但钟只在晚上敲响。红魔馆内部比外面看起来大得多。");
        builder.bulidString("主要成员：");
        builder.bulidItems(new String[] {
                           "蕾米莉亚·斯卡雷特（レミリア·スカーレット/Remilia Scarlet）",
                           "芙兰朵露·斯卡雷特（フランドール·スカーレット/Flandre Scarlet）",
                           "十六夜咲夜（いざよい さくや/Sakuya Izayoi）",
                           "帕秋莉·诺蕾姬（パチュリー·ノーレッジ/Patchouli Knowledge）",
                           "小恶魔（リートル デビッル/Little Devil）",
                           "红美铃（ホン メイリン/Hong Meirin）"
        });
        builder.bulidTitle("永远亭");
        builder.bulidString("永远亭是《东方project》中位于迷途竹林深处的一座日式建筑，是八意永琳、蓬莱山辉夜、因幡帝和铃仙·优昙华院·因幡的居住地。首次出现在《东方永夜抄》。");
        builder.bulidString("主要成员：");
        builder.bulidItems(new String[] {
                           "蓬莱山辉夜（ほうらいさん かぐや/Houraisan Kaguya）",
                           "八意永琳(やごころ えいりん/Eirin Yagokoro)",
                           "铃仙·优昙华院·因幡（レイセン・うどんげいん・イナバ/Reisen Udongein Inaba）",
                           "因幡帝（いなば てゐ/Tewi Inaba）"
        });
        builder.bulidTitle("白玉楼");
        builder.bulidString("白玉楼（はくぎょくろう/Hakugyokurou）是《东方project》中位于冥界的一座有着庭院的屋子，是西行寺幽幽子和魂魄妖梦的居住地。首次出现在《东方妖妖梦》。");
        builder.bulidString("主要成员：");
        builder.bulidItems(new String[] {
                           "西行寺幽幽子（さいぎょうじ　ゆゆこ/Saigyouji Yuyuko）",
                           "魂魄 妖梦(こんぱく　ようむ/Youmu Konpaku)"
        });
        builder.createFile(path + title);
    }
}
```

**Main类**

```
package design7;

import java.io.File;
import java.io.IOException;

public class Main {

    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException, IOException {
        String path = "D:" + File.separator;
        String title = "东方幻想乡";
        new Designer().construct(new TextBuilder(), path, title);
        new Designer().construct(new HtmlBuilder(), path, title);
    }
}
```

执行后，会在D盘根目录下生成东方幻想乡.txt及东方幻想乡.html两个文件。

东方幻想乡.txt的内容为：

```
====================东方幻想乡====================
#红魔馆#
红魔馆（こうまかん/Koumakan）是蕾米莉亚·斯卡雷特拥有的一间房子。它位于雾之湖的边缘。从外面可以看见深红色的窗户，但是窗户并不多，因为吸血鬼不喜欢阳光。里面也有很多没有窗户的房间。屋顶上有一个钟楼，但钟只在晚上敲响。红魔馆内部比外面看起来大得多。
主要成员：
    · 蕾米莉亚·斯卡雷特（レミリア·スカーレット/Remilia Scarlet）
    · 芙兰朵露·斯卡雷特（フランドール·スカーレット/Flandre Scarlet）
    · 十六夜咲夜（いざよい さくや/Sakuya Izayoi）
    · 帕秋莉·诺蕾姬（パチュリー·ノーレッジ/Patchouli Knowledge）
    · 小恶魔（リートル デビッル/Little Devil）
    · 红美铃（ホン メイリン/Hong Meirin）
#永远亭#
永远亭是《东方project》中位于迷途竹林深处的一座日式建筑，是八意永琳、蓬莱山辉夜、因幡帝和铃仙·优昙华院·因幡的居住地。首次出现在《东方永夜抄》。
主要成员：
    · 蓬莱山辉夜（ほうらいさん かぐや/Houraisan Kaguya）
    · 八意永琳(やごころ えいりん/Eirin Yagokoro)
    · 铃仙·优昙华院·因幡（レイセン・うどんげいん・イナバ/Reisen Udongein Inaba）
    · 因幡帝（いなば てゐ/Tewi Inaba）
#白玉楼#
白玉楼（はくぎょくろう/Hakugyokurou）是《东方project》中位于冥界的一座有着庭院的屋子，是西行寺幽幽子和魂魄妖梦的居住地。首次出现在《东方妖妖梦》。
主要成员：
    · 西行寺幽幽子（さいぎょうじ　ゆゆこ/Saigyouji Yuyuko）
    · 魂魄 妖梦(こんぱく　ようむ/Youmu Konpaku)

```

东方幻想乡.html的内容为：

```
<html>
  <head><title>东方幻想乡</title></head>
  <body>
    <h1>红魔馆</h1>
    <p>红魔馆（こうまかん/Koumakan）是蕾米莉亚·斯卡雷特拥有的一间房子。它位于雾之湖的边缘。从外面可以看见深红色的窗户，但是窗户并不多，因为吸血鬼不喜欢阳光。里面也有很多没有窗户的房间。屋顶上有一个钟楼，但钟只在晚上敲响。红魔馆内部比外面看起来大得多。</p>
    <p>主要成员：</p>
    <ul>
      <li>蕾米莉亚·斯卡雷特（レミリア·スカーレット/Remilia Scarlet）</li>
      <li>芙兰朵露·斯卡雷特（フランドール·スカーレット/Flandre Scarlet）</li>
      <li>十六夜咲夜（いざよい さくや/Sakuya Izayoi）</li>
      <li>帕秋莉·诺蕾姬（パチュリー·ノーレッジ/Patchouli Knowledge）</li>
      <li>小恶魔（リートル デビッル/Little Devil）</li>
      <li>红美铃（ホン メイリン/Hong Meirin）</li>
    </ul>
    <h1>永远亭</h1>
    <p>永远亭是《东方project》中位于迷途竹林深处的一座日式建筑，是八意永琳、蓬莱山辉夜、因幡帝和铃仙·优昙华院·因幡的居住地。首次出现在《东方永夜抄》。</p>
    <p>主要成员：</p>
    <ul>
      <li>蓬莱山辉夜（ほうらいさん かぐや/Houraisan Kaguya）</li>
      <li>八意永琳(やごころ えいりん/Eirin Yagokoro)</li>
      <li>铃仙·优昙华院·因幡（レイセン・うどんげいん・イナバ/Reisen Udongein Inaba）</li>
      <li>因幡帝（いなば てゐ/Tewi Inaba）</li>
    </ul>
    <h1>白玉楼</h1>
    <p>白玉楼（はくぎょくろう/Hakugyokurou）是《东方project》中位于冥界的一座有着庭院的屋子，是西行寺幽幽子和魂魄妖梦的居住地。首次出现在《东方妖妖梦》。</p>
    <p>主要成员：</p>
    <ul>
      <li>西行寺幽幽子（さいぎょうじ　ゆゆこ/Saigyouji Yuyuko）</li>
      <li>魂魄 妖梦(こんぱく　ようむ/Youmu Konpaku)</li>
    </ul>

```

用浏览器打开东方幻想乡.html的效果为：

![2.jpg](/images/blog_pic/Java 设计模式/7Builder模式/2.jpg)

# 登场角色

上面的示例程序介绍了Builder模式的Java实现，下面咱们试着跳出语言层面，抽象出Builder模式中登场的角色。

**Builder(建造者)**

在构造复杂事物时，Builder是每一个具体细节的制造者，但却不负责整体的规划。换句话说，如果我们把要构建的事物拆散了来看，每一个小零件其实都是Builder制造的，但是Builder并不知道它到底造了个什么，也没能力建造出一个完整的事物。

以前文的小例子来说，如果我们把最后输出的文章视为待构造的复杂事物的话，Builder负责生成一篇文章的所有细节：

```
public abstract void bulidHead(String head);

public abstract void bulidTitle(String title);

public abstract void bulidString(String str);

public abstract void bulidItems(String[] items);

public abstract void bulidTail();
```

但是只有这些是无法形成一篇文章的，关键是如何组合这些细节，以及填入什么内容。这些就是下面马上就要介绍的Designer的工作了。

在例子中，由Builder，TextBuilder及HtmlBuilder共同扮演了Builder这个角色。当然，这种将角色分层的做法并不是Builder模式所关注的核心。

**Designer(设计者)**

在构造复杂事物时，Designer负责整体的设计，并不负责具体每个零件的实现。具体实现会交给Builder。在例子中，Designer类扮演了Designer这个角色。

某些时候，我们也会将Designer与Builder合二为一，形成超级工人(也就是老板最喜欢的那种)：既能做设计，又能具体动手实现。不过这样会导致一个角色承载的功能过多，不利于程序架构的划分，在程序规模不大，不容易引起混乱时可以使用。

下面是抽象后，无关语言的类图：

![3.jpg](/images/blog_pic/Java 设计模式/7Builder模式/3.jpg)

# 相关设计模式

**[3.Template Method模式]()**
**[4.Factory Method模式]()**
**[8.Abstract Factory模式]()**

关于这4个设计模式间的联系与区别，详见[8.Abstract Factory模式]()。