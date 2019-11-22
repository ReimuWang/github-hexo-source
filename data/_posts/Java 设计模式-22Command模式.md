---
title: Java 设计模式-22.Command模式
date: 2018-09-07 10:10:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Command模式被归入了第10部分[用类来表现]()。在GoF原书中，Command模式则被归入了[行为型设计模式]()。

<!-- more -->

# 综述

在[19.State模式]()中，我们介绍了用类来描述状态。今天我们介绍另一个可以用类来描述的相对抽象的概念：命令。

宏观的说，一个类与其他类的关系可以归为两种：

- 向其他类发出命令促使其他类发生变化。
- 接到来自其他类的指令自身发生变化。

通常来说，系统只会记录变化的结果，而不会记录变化的过程。换句话说，当我们在某个时间点对系统进行查询时，通常只能查到系统在当前时间的静态状态，也就是因命令导致的结果，而无法查知变化的过程。虽然程序一般都会把这个过程记录在日志中，但从严格的意义上来讲，日志文件已经不属于系统本身了。

变化过程本身无法记录，但是如果我们将触发变化的命令作为对象记录下来呢？

假设我们要画一系列的点(这也是本文的示例程序，先简要描述下)，通常来说，我们会告诉绘图实例：在位置p1绘制一个半径为r1，颜色为c1的点！绘图实例就赶紧按要求画了一个点。我们再说：在位置p2绘制一个半径为r2，颜色为c2的点！绘图程序赶紧又画了一个。最终我们得到了绘制着两个点的画布。但是触发绘制这两个点的命令却丢失了：因为我们根本不曾记录。

如果我们将这两个命令记录下来，形成表示命令的实例：

- 命令1：在位置p1绘制一个半径为r1，颜色为c1的点！
- 命令2：在位置p2绘制一个半径为r2，颜色为c2的点！

这样我们便算是将一个抽象的变化存储为了相对具体的实体。一旦完成了存储，我们可做的事一下子就变多了：

1. 引入命令后，类A将从直接调用类B转化为制造一条命令，而后执行这条命令：这就使得类A与类B之间完成了解耦。
2. 接上条，这种解耦并不仅仅是空间上的，同样还可以是时间上的，如果对命令结果的需求没那么迫切，类A完全可以只负责制造命令，而后就可以继续做自己的事了。这使得命令的执行变成了异步的。
3. 继续接上条，一旦命令的执行变为异步，就意味着我们可以把待执行的命令存储起来，形成一个指令集。在需要的时候统一执行。
4. [18.Memento模式]()可以让我们进行存档与恢复。其做法就是把某个时间点的对象像拍照那样存储下来，而后在需要时再进行恢复；引入命令后，我们也可以做到类似的事，比如某实例最初处于初始状态，而后经过了命令1，2，3后变为了状态2。如果我们想要记录状态2，方法之一自然是使用[18.Memento模式]()将状态2照下来。但我们同样也可以存储命令1，2，3。当我们需要状态2时，我们只需要再找一个处于初始状态的实例，而后再按顺序执行一遍命令1，2，3即可。
5. 接上条，很显然，如果要使用命令集来制作存档的话，在恢复存档时通常要比拍快照更花时间。

将这种"引入命令"的思想理论化后，得到的就是Command模式啦。有时，我们也会将命令称为事件(event)，它与"事件驱动编程"中的"事件"的含义是相同的。我们会在GUI编程中大量的用到事件：点击一次鼠标是一个事件，按下键盘上的一个按键是一个事件。而每个事件其实都可以视为一个指令，以促使系统发生相应的变化。

# 示例程序需求分析

下面我们就来设计一个应用了Command模式的小例子吧。这是一个使用Java Swing技术实现的画图小程序。启动时的初始面板为：

![0.jpg](/images/blog_pic/Java 设计模式/22Command模式/0.jpg)

区域介绍：

![1.jpg](/images/blog_pic/Java 设计模式/22Command模式/1.jpg)

默认画笔粗细为10(画笔形成的点的半径为10个像素)，默认颜色为黑色，在画布上按住鼠标左键拖动即可绘制图形。下图为使用初始参数绘制一条线：

![2.jpg](/images/blog_pic/Java 设计模式/22Command模式/2.jpg)

然后我们调整画笔粗细为5，颜色为绿，在线的下面再画一个圈：

![3.jpg](/images/blog_pic/Java 设计模式/22Command模式/3.jpg)

点击"清空"按钮后，画布会被清空。不过画笔颜色及粗细不会恢复为默认值。例如我们可以撤销部分绘制圆的操作：

![4.jpg](/images/blog_pic/Java 设计模式/22Command模式/4.jpg)

点击"撤销"按钮可以让我们撤销一次操作，即取消绘制一个点。不过并不会撤销对画笔颜色及粗细的选择。

点击"保存"按钮可以将当前画布上的图形保存到文件中。此后可以通过点击"读取"按钮恢复此前保存的图形。

功能基本就是这么多，最后附上一幅我用这个程序画的一幢好丑好丑的房子，哈哈哈：

![5.jpg](/images/blog_pic/Java 设计模式/22Command模式/5.jpg)

# 示例程序

本程序中的所有代码将被统一置于design22包下，结构如下：

![6.jpg](/images/blog_pic/Java 设计模式/22Command模式/6.jpg)

下面将逐个贴出每个类的源码。

首先介绍command包里的类，顾名思义，这个包下自然全都是命令啦。

**Command接口**

```
package design22.command;

import java.util.List;

public interface Command {

    void execute();

    List<String> strList();
}
```

Command是最顶层的命令接口。内部只有两个方法。其中execute()表示执行命令。而strList()则会将命令转换为字符串以用于存档。之所以返回类型是List，是因为实际实现Command接口的命令有可能并非单独一条指令，而是一个指令集。

**MacroCommand类**

```
package design22.command;

import java.awt.Color;
import java.awt.Point;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Stack;

import org.apache.commons.lang.StringUtils;

import com.alibaba.fastjson.JSONObject;

import design22.view.MyCanvas;

public class MacroCommand implements Command {

    private Stack<Command> commands = new Stack<Command>();

    private static final String SAVE = "src/main/java/design22/save/save.txt";

    private static final String ENCODING = "UTF-8";

    private MyCanvas canvas;

    public MacroCommand(MyCanvas canvas) {
        this.canvas = canvas;
    }

    @Override
    public void execute() {
        Iterator<Command> iterator = this.commands.iterator();
        while (iterator.hasNext()) iterator.next().execute();
    }

    public void append(Command cmd) {
        if (this != cmd) this.commands.push(cmd);
    }

    public void clear() {
        this.commands.clear();
    }

    public void undo() {
        if (!this.commands.isEmpty()) this.commands.pop();
    }

    public void save() {
        List<String> data = this.strList();
        FileOutputStream fo = null;
        OutputStreamWriter ow = null;
        BufferedWriter bw = null;
        try {
            fo = new FileOutputStream(new File(MacroCommand.SAVE));
            ow = new OutputStreamWriter(fo, MacroCommand.ENCODING);
            bw = new BufferedWriter(ow);
            for (String str : data) {
                bw.write(str);
                bw.newLine();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                bw.close();
                ow.close();
                fo.close();
            } catch (Exception ef) {
                ef.printStackTrace();
            }
        }
    }

    @Override
    public List<String> strList() {
        List<String> list = new ArrayList<String>();
        Iterator<Command> iterator = this.commands.iterator();
        while (iterator.hasNext()) list.addAll(iterator.next().strList());
        return list;
    }

    public void load() {
        this.clear();
        this.canvas.init();
        FileInputStream fi = null;
        InputStreamReader ir = null;
        BufferedReader br = null;
        try {
            fi = new FileInputStream(new File(MacroCommand.SAVE));
            ir = new InputStreamReader(fi, MacroCommand.ENCODING);
            br = new BufferedReader(ir);
            String lineTxt = null;
            while (!StringUtils.isBlank((lineTxt = br.readLine()))) {
                JSONObject jo = JSONObject.parseObject(lineTxt);
                String className = jo.get("type").toString();
                Command cmd = null;
                if (DrawCommand.class.getName().equals(className)) {
                    cmd = new DrawCommand(this.canvas, new Point(Integer.parseInt(jo.get("pointX").toString()), Integer.parseInt(jo.get("pointY").toString())));
                } else if (ColorCommand.class.getName().equals(className)) {
                    cmd = new ColorCommand(this.canvas, new Color(Integer.parseInt(jo.get("colorRGB").toString())));
                } else if (BrushCommand.class.getName().equals(className)) {
                    cmd = new BrushCommand(this.canvas, Integer.parseInt(jo.get("brushWidth").toString()));
                }
                if (null != cmd) this.append(cmd);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                br.close();
                ir.close();
                fi.close();
            } catch (Exception ef) {
                ef.printStackTrace();
            }
        }
    }
}
```

MacroCommand类就是前文提到的，实现了Command接口的指令集。其最核心的字段就是commands了。对于这个字段，我们有两点需要说明：

1. 之所以声明为栈，是为了便于进行撤销操作。
2. 栈的泛型类型是Command，而MacroCommand类本身又实现了Command接口。这意味着commands字段中同样可以添加MacroCommand类型的数据。也就是说，指令集中的某条指令同样可以是另一个指令集，这在MacroCommand类execute()中也有所体现：最终我们是使用递归执行栈中的所有指令的(为防止无限递归下去，我们禁止将自身添加到commands字段中)。这其实是对[11.Composite模式]()的一种应用。

commands是本程序中最核心的字段，其中存储了所有的指令。程序做的所有操作最终其实都是在操作这个字段：

- 清空画布：即清空commands字段。而后按照commands字段重绘图形，因为commands已被清空，自然就起到了清空画布的效果。
- 撤销最后一次操作：弹出最后一条指令，而后按照commands字段重绘图形，因为最后一条指令已被弹出，自然就相当于撤销操作了。
- 保存：将commands中的命令按顺序存储入文件。
- 读取：将文件中的命令按顺序读取入commands字段，而后依commands重绘图形，起到读档的作用。

**DrawCommand类**

```
package design22.command;

import java.awt.Point;
import java.util.ArrayList;
import java.util.List;

import com.alibaba.fastjson.JSONObject;

import design22.view.MyCanvas;

public class DrawCommand implements Command {

    private MyCanvas canvas;

    private Point point;

    public DrawCommand(MyCanvas canvas, Point point) {
        this.canvas = canvas;
        this.point = point;
    }

    @Override
    public void execute() {
        this.canvas.draw(this.point);
    }

    @Override
    public List<String> strList() {
        List<String> list = new ArrayList<String>(1);
        JSONObject jo = new JSONObject();
        jo.put("type", DrawCommand.class.getName());
        jo.put("pointX", this.point.x);
        jo.put("pointY", this.point.y);
        list.add(jo.toJSONString());
        return list;
    }
}
```

命令：在画布特定的位置上绘制一个点。

**ColorCommand类**

```
package design22.command;

import java.awt.Color;
import java.util.ArrayList;
import java.util.List;

import com.alibaba.fastjson.JSONObject;

import design22.view.MyCanvas;

public class ColorCommand implements Command {

    private MyCanvas canvas;

    private Color color;

    public ColorCommand(MyCanvas canvas, Color color) {
        this.canvas = canvas;
        this.color = color;
    }

    @Override
    public void execute() {
        this.canvas.setColor(this.color);
    }

    @Override
    public List<String> strList() {
        List<String> list = new ArrayList<String>(1);
        JSONObject jo = new JSONObject();
        jo.put("type", ColorCommand.class.getName());
        jo.put("colorRGB", this.color.getRGB());
        list.add(jo.toJSONString());
        return list;
    }
}
```

命令：将画笔变更为特定的颜色。

**BrushCommand类**

```
package design22.command;

import java.util.ArrayList;
import java.util.List;

import com.alibaba.fastjson.JSONObject;

import design22.view.MyCanvas;

public class BrushCommand implements Command {

    private MyCanvas canvas;

    private int width;

    public BrushCommand(MyCanvas canvas, int width) {
        this.canvas = canvas;
        this.width = width;
    }

    @Override
    public void execute() {
        this.canvas.setR(this.width);
    }

    @Override
    public List<String> strList() {
        List<String> list = new ArrayList<String>(1);
        JSONObject jo = new JSONObject();
        jo.put("type", BrushCommand.class.getName());
        jo.put("brushWidth", this.width);
        list.add(jo.toJSONString());
        return list;
    }
}
```

命令：将画笔变更为特定的粗细。

然后是view包：

**MyCanvas类**

```
package design22.view;

import java.awt.Canvas;
import java.awt.Color;
import java.awt.Graphics;
import java.awt.Point;

import design22.command.MacroCommand;

public class MyCanvas extends Canvas {

    private static final long serialVersionUID = 1L;

    private static final Color DEF_COLOR = Color.BLACK;

    private static final int DEF_R = 10;

    Color color = MyCanvas.DEF_COLOR;

    int r = MyCanvas.DEF_R;

    MacroCommand history = new MacroCommand(this);

    public MyCanvas(int width, int height) {
        this.setSize(width, height);
        this.setBackground(Color.WHITE);
    }

    public void paint(Graphics g) {
        this.init();
        this.history.execute();
    }

    public void draw(Point point) {
        Graphics g = this.getGraphics();
        g.setColor(this.color);
        g.fillOval(point.x - this.r, point.y - this.r, 2 * this.r, 2 * this.r);
    }

    public void setColor(Color color) {
        this.color = color;
    }

    public void setR(int r) {
        this.r = r;
    }

    public void init() {
        this.color = MyCanvas.DEF_COLOR;
        this.r = MyCanvas.DEF_R;
    }
}
```

即为自己实现的画板类。

**View类**

```
package design22.view;

import java.awt.Color;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.MouseEvent;
import java.awt.event.MouseMotionAdapter;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.util.ArrayList;
import java.util.List;

import javax.swing.Box;
import javax.swing.BoxLayout;
import javax.swing.ButtonGroup;
import javax.swing.JButton;
import javax.swing.JCheckBox;
import javax.swing.JFrame;

import design22.command.BrushCommand;
import design22.command.ColorCommand;
import design22.command.Command;
import design22.command.DrawCommand;

public class View extends JFrame implements ActionListener {

    private static final long serialVersionUID = 1L;

    MyCanvas canvas = new MyCanvas(50 * 16, 50 * 9);

    private int maxBrush = 20;

    private List<JCheckBox> brushBoxs = new ArrayList<JCheckBox>();

    private List<ColorBox> colorBoxs = new ArrayList<ColorBox>();

    private List<JButton> buttons = new ArrayList<JButton>();

    public View() {
        // 制作按钮
        this.buttons.add(new JButton("清空"));
        this.buttons.add(new JButton("撤销"));
        this.buttons.add(new JButton("保存"));
        this.buttons.add(new JButton("读取"));
        for (JButton button : this.buttons) button.addActionListener(this);
        // 制作颜色CheckBox
        this.colorBoxs.add(new ColorBox(Color.BLACK, "黑", this));
        this.colorBoxs.add(new ColorBox(Color.DARK_GRAY, "深灰", this));
        this.colorBoxs.add(new ColorBox(Color.GRAY, "灰", this));
        this.colorBoxs.add(new ColorBox(Color.LIGHT_GRAY, "浅灰", this));
        this.colorBoxs.add(new ColorBox(Color.BLUE, "蓝", this));
        this.colorBoxs.add(new ColorBox(Color.CYAN, "青", this));
        this.colorBoxs.add(new ColorBox(Color.GREEN, "绿", this));
        this.colorBoxs.add(new ColorBox(Color.MAGENTA, "洋红", this));
        this.colorBoxs.add(new ColorBox(Color.RED, "红", this));
        this.colorBoxs.add(new ColorBox(Color.PINK, "粉", this));
        this.colorBoxs.add(new ColorBox(Color.ORANGE, "橘", this));
        this.colorBoxs.add(new ColorBox(Color.YELLOW, "黄", this));
        ButtonGroup colorGroup = new ButtonGroup();
        for (ColorBox colorBox : this.colorBoxs) {
            colorBox.checkBox.addActionListener(this);
            colorGroup.add(colorBox.checkBox);
        }
        // 制作画笔CheckBox
        ButtonGroup brushGroup = new ButtonGroup();
        for (int i = 1; i <= this.maxBrush; i++) {
            JCheckBox brushCheckBox = new JCheckBox(i + "", i == this.canvas.r);
            this.brushBoxs.add(brushCheckBox);
            brushCheckBox.addActionListener(this);
            brushGroup.add(brushCheckBox);
        }
        // 画布拖动监听
        this.canvas.addMouseMotionListener(
            new MouseMotionAdapter() {
                @Override
                public void mouseDragged(MouseEvent e) {
                    Command cmd = new DrawCommand(View.this.canvas, e.getPoint());
                    View.this.canvas.history.append(cmd);
                    cmd.execute();
                }
            }
        );
        // 布局
        Box firstBox = new Box(BoxLayout.X_AXIS);
        for (JButton button : this.buttons) firstBox.add(button);
        Box secondBox = new Box(BoxLayout.X_AXIS);
        Box brushBox = new Box(BoxLayout.Y_AXIS);
        for (JCheckBox temp : this.brushBoxs) brushBox.add(temp);
        secondBox.add(brushBox);
        secondBox.add(this.canvas);
        Box colorBox = new Box(BoxLayout.Y_AXIS);
        for (ColorBox temp : this.colorBoxs) colorBox.add(temp.checkBox);
        secondBox.add(colorBox);
        Box mainBox = new Box(BoxLayout.Y_AXIS);
        mainBox.add(firstBox);
        mainBox.add(secondBox);
        super.getContentPane().add(mainBox);
        // 关闭
        this.addWindowListener(
            new WindowAdapter() {
                @Override
                public void windowClosing(WindowEvent e) {
                    System.exit(0);
                }
            }
        );
        // 显示
        super.pack();
        super.setVisible(true);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        for (JButton button : this.buttons) {
            if (e.getSource() == button) {
                if ("清空".equals(button.getText())) {
                    this.canvas.history.clear();
                    this.canvas.repaint();
                } else if ("撤销".equals(button.getText())) {
                    this.canvas.history.undo();
                    this.canvas.repaint();
                } else if ("保存".equals(button.getText())) {
                    this.canvas.history.save();
                } else if ("读取".equals(button.getText())) {
                    this.canvas.history.load();
                    this.canvas.repaint();
                }
                return;
            }
        }
        for (ColorBox colorBox : this.colorBoxs) {
            if (e.getSource() == colorBox.checkBox) {
                Command cmd = new ColorCommand(this.canvas, colorBox.color);
                this.canvas.history.append(cmd);
                cmd.execute();
                return;
            }
        }
        for (JCheckBox brushBox : this.brushBoxs) {
            if (e.getSource() == brushBox) {
                Command cmd = new BrushCommand(this.canvas, Integer.parseInt(brushBox.getText()));
                this.canvas.history.append(cmd);
                cmd.execute();
                return;
            }
        }
    }
}

class ColorBox {

    Color color;

    JCheckBox checkBox;

    String desc;

    ColorBox (Color color, String desc, View view) {
        this.color = color;
        this.checkBox = new JCheckBox(desc, this.color == view.canvas.color);
    }
}
```

View负责创建及调用命令。方式主要有两种。

第一种为创建命令并直接调用。以切换颜色为例，会触发actionPerformed()监听事件。从代码中可以看到，我们会创建并直接执行命令。当然，我们也会将该命令存入MacroCommand类的commands字段中，以形成存档，View作为面板，每个实例都会创建并绑定一个MyCanvas(画布)类的实例，而每一个画布，同样会创建并绑定一个MacroCommand类的实例：在MyCanvas中，这个MacroCommand被称为history。

第二种并不会创建新命令，但会导致commands中的已有命令会被重新全部执行一遍。例如当我们按下"撤销"按钮后，会触发actionPerformed()方法。其中undo()会导致commands中最新的命令被弹出，而repaint()则会先清空画布，而后将commands中剩余的命令重新执行一遍。

**save.txt**

我们会在save包下存放存档。该存档只有一份，反复存档只会导致存档覆盖。因为拖动时每一个点都会生成1条命令，因此这个文件行数通常都会很多。上文绘制房子的那个存档文件就有16000+的行数。现只截取最开始的那部分：

```
{"brushWidth":20,"type":"design22.command.BrushCommand"}
{"colorRGB":-16711936,"type":"design22.command.ColorCommand"}
{"pointX":4,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":6,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":8,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":10,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":13,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":16,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":20,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":25,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":29,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":32,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":35,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":37,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":40,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":42,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":45,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":48,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":52,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":54,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":56,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":60,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":63,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":67,"pointY":411,"type":"design22.command.DrawCommand"}
{"pointX":72,"pointY":412,"type":"design22.command.DrawCommand"}
```

最后是直接位于design22包下的Main.java：

```
package design22;

import design22.view.View;

public class Main {

    public static void main(String[] args) {
        new View();
    }
}
```

执行该类后，即可启动程序。

# 登场角色

上面的示例程序介绍了Command模式的Java实现，下面咱们试着跳出语言层面，抽象出Command模式中登场的角色。

**Command(命令)**

在示例程序中，由Command接口扮演这个角色。

**ConcreteCommand(具体的命令)**

在示例程序中，由MacroCommand类，DrawCommand类，ColorCommand类，BrushCommand类联袂扮演这个角色。

**Receiver(接收者)**

接收命令，或是受命令影响的角色。在示例程序中，由MyCanvas类扮演这个角色。

**Invoker(发动者)**

Invoker是Command的触发者(通常也是创建者)。在示例程序中，由View类扮演这个角色。

# 适配器

在此，我想说点与Command模式没什么关系的事。

在示例程序中，View实现监听的方式是写匿名的适配器类。例如我们在为画布添加拖动鼠标的监听时，是这样做的：

```
this.canvas.addMouseMotionListener(
    new MouseMotionAdapter() {
        @Override
        public void mouseDragged(MouseEvent e) {
            Command cmd = new DrawCommand(View.this.canvas, e.getPoint());
            View.this.canvas.history.append(cmd);
            cmd.execute();
        }
    }
);
```

我们不妨稍微看下源码。这个this.canvas.addMouseMotionListener()的方法定义为：

```
public synchronized void addMouseMotionListener(MouseMotionListener l)
```

很显然，它接收一个MouseMotionListener类型的数据，而这个MouseMotionListener则是一个接口：

```
package java.awt.event;

import java.util.EventListener;

public interface MouseMotionListener extends EventListener {

    public void mouseDragged(MouseEvent e);

    public void mouseMoved(MouseEvent e);

}
```

上面是MouseMotionListener接口的全部源码。其中mouseDragged()表示"按住鼠标拖动"这一事件；而mouseMoved()则表示鼠标移动事件。

那么自然，我们传入的这个MouseMotionAdapter就该实现MouseMotionListener啦，那么是不是呢：

```
package java.awt.event;

public abstract class MouseMotionAdapter implements MouseMotionListener {

    public void mouseDragged(MouseEvent e) {}

    public void mouseMoved(MouseEvent e) {}
}
```

代码不长，我就全部贴出了。可以看到，MouseMotionAdapter除了将接口中约束的两个方法声明非抽象的以外，实际上什么都没做。

其实比起这种使用匿名内部类，也就是适配器的做法，我们其实可以直接找一个类实现MouseMotionListener接口，然后将这个类的实例传给addMouseMotionListener()方法(通常来说，我们会让面板去实现这个接口，也就是说传入this)。那么二者各有什么利弊呢？

先说说使用匿名类的好处吧。如果要自己写一个类去实现监听接口，这就意味着必须要强制实现接口中所有约束的方法。例如，此时我们就必须实现mouseMoved()了，即便我们根本不关心鼠标移动事件，那也要写一个空方法放在那里才行。这在接口本身约束很多(例如WindowListener)，而实际用到的很少时尤其的麻烦：我们需要写大量的空方法，使得代码有失优雅。而使用适配器类则解决了这个问题。MouseMotionAdapter被声明为了抽象类，但其中并没有抽象方法。这样做的原因有二：类被声明为抽象的是不希望直接new出对象来，要写继承的子类才行；而方法均不是抽象的则让自己只需要重写需要的约束即可。

然后说一下继承接口的好处。使用匿名类的问题在于，每次使用都会初始化一个新的匿名类，在需要多次使用同一个监听时将会很不方便。如果专门弄一个类去监听，又显得小题大做。当然，直接让面板或其他组件更是糟糕的决策：从使用上讲，因为Java是单继承的语言，这会消耗掉宝贵的继承机会；另一方面，组件根本就不是监听器，也就是根本不符合里氏替换原则及合成聚合复用原则，从逻辑上更是说不通。因此，此时让面板直接实现接口就成为了最好的选择：View便直接实现了ActionListener，而后将需要此监听的按钮及选框均与其绑定。