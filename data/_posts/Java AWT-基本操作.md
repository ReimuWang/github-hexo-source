---
title: Java AWT-基本操作
date: 2018-02-03 12:37:55
tags: [Java,GUI,AWT]
categories: Java AWT
---

# 创建基本的窗体

AWT中封装窗体逻辑的类为java.awt.Frame，通常我们通过继承它来使用它的功能，它的类定义为：

```
package java.awt;

public class Frame extends Window implements MenuContainer
```

<!-- more -->

下面我们给出Frame常用方法的定义。首先给出Frame继承自它的父类java.awt.Window的方法：

```
/**
 * 设定窗体相对于屏幕左上角的位置为(x,y)
 */
public void setLocation(int x, int y)

/**
 * 设定窗体的宽为width，高为height
 */
public void setSize(int width, int height)
```

然后我们就可以开始写第一个小例子了：

```
import java.awt.Frame;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
    }

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```

运行后无输出，程序也无法终止。之所以无输出，是因为窗体默认是不可见的。而之所以无法终止，是因为窗体虽然是不可见的，但它的确存在。

为了让窗体可见，我们还要使用Frame继承自它的父类java.awt.Window中的另一个方法：

```
/**
 * true--窗体可见,false--窗体不可见
 * 默认为false
 */
public void setVisible(boolean b)
```

相应的代码改为：

```
import java.awt.Frame;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
        super.setVisible(true);
    }

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```

此时运行程序，会按设定生成所需的窗体：

![0.jpg](/images/blog_pic/Java AWT/基本操作/0.jpg)

之所以不默认设定窗体可见，是为了使得窗体可见的时机可控。很多时候，窗体的加载是一个耗时的操作，我们并不希望给用户展示一个"半成品"，此时我们就可以待窗体加载到"可以见人了"的程度后再设定为可见。

至此我们已绘制出了窗体，不过当我们点击窗体右上角的叉时却无法关闭窗体，其原因为默认情况下并没有给"点击叉"这一行为关联任何动作。因此我们需要自行添加窗口监听设定被触发的行为。添加窗口监听的方法依然在java.awt.Window中：

```
public synchronized void addWindowListener(WindowListener l)
```

不妨稍微插一句，至此为止我们介绍的所有Frame中的方法都不是存在于Frame中的，而是Frame继承自其父类Window的。事实上，我们后文介绍的方法也基本都是这个套路。核心的骨架已在Window中确定好，Frame只负责封装一些小的细节。

引入监听器后，代码改进为：

```
import java.awt.Frame;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
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

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```

这里我们让"点击叉"这个行为触发程序的结束。现在我们再运行程序，就可以通过点击右上角的叉结束程序，关闭窗体了。之所以不将结束程序设定为默认操作，是为了给程序员留出编码空间，例如，我们可以在真正关闭前设定再弹出一个框:"是否确定退出？"做二次确认。事实上，绝大多数的桌面应用软件也都是这么做的。

# 绘制窗体中的组件

绘制方法依然在java.awt.Window中：

```
public void paint(Graphics g)
```

其中Graphics可以理解为画笔。通常我们通过重写该方法来封装自身的业务逻辑：

```
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
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
    }

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```

运行后并未绘制任何组件：这是理所当然的，因为paint()方法为空。下面我们就来举一个绘制的小例子：

```
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
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
        g.drawLine(100, 100, 200, 200);
    }

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```

输出：

![1.jpg](/images/blog_pic/Java AWT/基本操作/1.jpg)

这样就绘制了一条线段。由此可见，绘制组件最终是要着落在Graphics这个类上的。

# Graphics绘制基本图形

下面我们会介绍Graphics类常用的绘制基本图形的方法，基本沿用前文代码，若有需要可能会改变窗体大小。

首先是上一小节已经展示的：

```
/**
 * 绘制一条线段
 * @param x1 int, 端点1横坐标
 * @param y1 int, 端点1纵坐标
 * @param x2 int, 端点2横坐标
 * @param y2 int, 端点2纵坐标
 */
public abstract void drawLine(int x1, int y1, int x2, int y2)
```

---

```
/**
 * 绘制一个矩形
 * @param x int, 左上角横坐标
 * @param y int, 左上角纵坐标
 * @param width int, 矩形宽
 * @param height int, 矩形高
 */
public void drawRect(int x, int y, int width, int height)
```

实例：

```
g.drawRect(0, 0, 100, 100);
```

输出：

![2.jpg](/images/blog_pic/Java AWT/基本操作/2.jpg)

上一个绘制线段的例子可能看得不够真切，不过本例中就暴露了一个需要注意的点：绘制图形时会以窗体的外轮廓为起点，并不会让出标题栏的宽度。

---

```
/**
 * 绘制一个椭圆
 * 给定的入参实质上是在确定椭圆所属的外切矩形
 * @param x int, 左上角横坐标
 * @param y int, 左上角纵坐标
 * @param width int, 矩形宽
 * @param height int, 矩形高
 */
public abstract void drawOval(int x, int y, int width, int height)
```

示例：

```
g.drawOval(100, 100, 100, 200);
```

输出：

![3.jpg](/images/blog_pic/Java AWT/基本操作/3.jpg)

特别的，若外切矩形是一个正方形，绘制出的自然是一个圆了，示例：

```
g.drawOval(100, 100, 200, 200);
```

输出：

![4.jpg](/images/blog_pic/Java AWT/基本操作/4.jpg)

---

```
/**
 * 绘制文字
 * @param x String, 文字内容
 * @param x int, 第一个字符的横坐标
 * @param y int, 字符串的纵坐标
 */
public abstract void drawString(String str, int x, int y)
```

示例：

```
g.drawString("博丽灵梦", 0, 100);
```

输出：

![5.jpg](/images/blog_pic/Java AWT/基本操作/5.jpg)

字符串并没有打全，其原因就在于其实不仅仅是标题栏，整个窗体其实围了一圈的外边框，而这一圈外边框都是无法绘制图形的，但是计算坐标又是以窗体外边框为起点。不得不说，这可着实让人蛋疼。

如果想改变字体，可自定义字体类：

```
g.setFont(new Font("楷体", Font.BOLD, 20));
g.drawString("博丽灵梦", 100, 100);
```

输出：

![6.jpg](/images/blog_pic/Java AWT/基本操作/6.jpg)

---

```
/**
 * 绘制一个填充矩形
 * @param x int, 左上角横坐标
 * @param y int, 左上角纵坐标
 * @param width int, 矩形宽
 * @param height int, 矩形高
 */
public abstract void fillRect(int x, int y, int width, int height)
```

示例：

```
g.fillRect(100, 100, 100, 200);
```

输出：

![7.jpg](/images/blog_pic/Java AWT/基本操作/7.jpg)

同理：

```
public abstract void fillOval(int x, int y, int width, int height)
```

则可绘制一个填充椭圆。

# Graphics改变画笔颜色

在绘制前修改画笔颜色即可：

```
g.setColor(Color.BLUE);
g.fillRect(100, 100, 100, 200);
```

输出：

![8.jpg](/images/blog_pic/Java AWT/基本操作/8.jpg)

为避免画笔颜色被修改乱套，建议每次修改前保存下，用完后再改回去：

```
Color base = g.getColor();
g.setColor(Color.BLUE);
g.fillRect(100, 100, 100, 200);
g.setColor(base);
```

# Graphics绘制图片

事实上，绘制基本图形的方法上文只介绍了其中很小的一本分，之所以不都介绍完，是因为实际写代码时基本用不到，而用不到的原因则在于：

**实在是太TM丑了！**

实际编程中，我们基本只会绘制一种组件：那就是图片。

既然要绘制图片，第一步自然就是将图片加载至内存：

```
/**
 * 加载默认图片目录下的图片
 * 图片默认目录为：src/main/resources/images
 * @param path String, 图片相对于图片默认目录的路径。
 *             例如图片为src/main/resources/images/a.jpg
 *             则path=a.jpg
 * @return Image, 存入内存中的图片信息
 * @throws IOException 
 */
public static Image loadImage(String path) throws IOException {
    URL url = Utils.class.getClassLoader()
                         .getResource("images/" + path);
    BufferedImage bImage = ImageIO.read(url);
    return bImage;
}
```

测试用图命名为test.jpg，丢到images目录下。test.jpg为：

![9.jpg](/images/blog_pic/Java AWT/基本操作/9.jpg)

然后我们就可以开始绘制了，此时的paint方法为：

```
@Override
public void paint(Graphics g) {
    try {
        Image image = Utils.loadImage("test.jpg");
        g.drawImage(image, 100, 100, null);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

其中用于绘制的方法的定义为：

```
/**
 * 绘制图片
 * @param img Image, 加载至内存中的图片
 * @param x int, 目标位置左上角横坐标
 * @param y int, 目标位置左上角纵坐标
 * @param observer ImageObserver, 通常传入null即可
 */
public abstract boolean drawImage(Image img, int x, int y, ImageObserver observer)
```

输出为：

![10.jpg](/images/blog_pic/Java AWT/基本操作/10.jpg)

很显然，该方法取了源图片全图，并且不加任何修改。如果只想取部分图片，或是调整图片大小比例，可使用如下方法：

```
/**
 * 绘制图片
 * @param img Image, 加载至内存中的图片
 * @param dx1 int, 目标图片左上角横坐标
 * @param dy1 int, 目标图片左上角纵坐标
 * @param dx2 int, 目标图片右下角横坐标
 * @param dy2 int, 目标图片右下角纵坐标
 * @param sx1 int, 源图片左上角横坐标
 * @param sy1 int, 源图片左上角纵坐标
 * @param sx2 int, 源图片右下角横坐标
 * @param sy2 int, 源图片右下角纵坐标
 * @param observer ImageObserver, 通常传入null即可
 */
public abstract boolean drawImage(Image img,
                                  int dx1, int dy1, int dx2, int dy2,
                                  int sx1, int sy1, int sx2, int sy2,
                                  ImageObserver observer);
```

乍一看入参可真他娘的多啊，好麻烦的样子。仔细看来其实很有条理：(sx1,sy1)-(sx2,sy2)确定了源图片的取图区域，(dx1,dy1)-(dx2,dy2)确定了目标位置的放置区域。这两份区域大小，比例可不相同，进而使得绘制出的图片的大小，比例不同于源图。

示例：

```
g.drawImage(image
           , 17, 50, 517, 550
           , 400, 150, 900, 650
           , null); 
```

输出：

![11.jpg](/images/blog_pic/Java AWT/基本操作/11.jpg)

# 绘制通用控件

如果要绘制按钮，文本输入框，选择框之类的控件，可参见[Java 设计模式-16.Mediator模式]()中的示例程序。