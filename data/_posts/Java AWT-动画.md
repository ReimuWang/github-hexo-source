---
title: Java AWT-动画
date: 2018-02-03 16:55:55
tags: [Java,GUI,AWT]
categories: Java AWT
---

我们先创建基本的窗体：

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

<!-- more -->

所谓动画，就是快速的绘制不同的静态画面，让人看起来"画动了"。AWT中绘制组件的流程为：

```
repaint() —> update() —> paint()
```

要实现动画，最简单的方式就是写一个线程，然后高频率的repaint()即可。

则上述程序可改造为：

```
import java.awt.Font;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
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
        g.drawString("" + (System.currentTimeMillis() / 100), 100, 100);
    }

    private class RepaintRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    GameFrame.this.repaint();
                    Thread.sleep(40);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```

每隔40ms刷新一次，为体现刷新效果，我们绘制了秒数随时间的变化。运行后，可以观测到秒数确实在按要求变动，但闪烁情况极其严重。

这是为什么呢？前文我们已经提到，AWT中绘制组件的流程为：

```
repaint() —> update() —> paint()
```

我们可以看一下update()的代码：

```
public void update(Graphics g) {
    if (isShowing()) {
        if (! (peer instanceof LightweightPeer)) {
            g.clearRect(0, 0, width, height);
        }
        paint(g);
    }
}
```

很明显，所谓的更新，其实是首先进行了清屏，也就是g.clearRect()，随后再重新画一遍。不断的清了画清了画，自然会导致闪烁。

既然找到了原因，那么解决思路就很明确了，自然是要重写update()方法才行：

```
import java.awt.Font;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
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
        g.drawString("" + (System.currentTimeMillis() / 100), 100, 100);
    }

    private class RepaintRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    GameFrame.this.repaint();
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

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```

此时闪烁情况得到了极大的缓解，已无法被人眼所见。

下面我们再来看一个小例子，实现一个匀速直线运动，遇到边框会反弹的小球：

```
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    private int x = 100;

    private int y = 100;

    private boolean ifXPositive = true;

    private boolean ifYPositive = true;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
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
        final int titleWidth = 40;    // 标题栏宽度
        final int padding = 10;    // 左右下留白宽度
        final int diameter = 20;    // 小球直径
        final int xLength = 3;    // 距上次刷新x轴移动距离的绝对值
        final int yLength = 3;    // 距上次刷新y轴移动距离的绝对值
        g.fillOval(this.x, this.y, diameter, diameter);
        this.x = this.ifXPositive ? this.x + xLength : this.x - xLength;
        this.y = this.ifYPositive ? this.y + yLength : this.y - yLength;
        if (this.x <= padding) {
            this.x = padding;
            this.ifXPositive = !this.ifXPositive;
        }
        if (this.x >= this.getWidth() - padding - diameter) {
            this.x = this.getWidth() - padding - diameter;
            this.ifXPositive = !this.ifXPositive;
        }
        if (this.y <= titleWidth) {
            this.y = titleWidth;
            this.ifYPositive = !this.ifYPositive;
        }
        if (this.y >= this.getHeight() - padding - diameter) {
            this.y = this.getHeight() - padding - diameter;
            this.ifYPositive = !this.ifYPositive;
        }
    }

    private class RepaintRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    GameFrame.this.repaint();
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

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```

上例中小球将永久运动下去，我们也可以在小球的运动过程中让其逐渐损失速度，最终停止：

```
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class GameFrame extends Frame {

    private static final long serialVersionUID = 1L;

    private int x = 100;

    private int y = 100;

    private boolean ifXPositive = true;

    private boolean ifYPositive = true;

    /**
     * int, 距上次刷新x轴移动距离的绝对值
     */
    private int xLength = 300;

    /**
     * int, 距上次刷新y轴移动距离的绝对值
     */
    private int yLength = 300;

    public void launchFrame() {
        super.setLocation(700, 300);
        super.setSize(400, 300);
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
        final int titleWidth = 40;    // 标题栏宽度
        final int padding = 10;    // 左右下留白宽度
        final int diameter = 20;    // 小球直径
        final double decay = 0.99999;    // 两次时间间隔间的速度衰减
        g.fillOval(this.x, this.y, diameter, diameter);
        this.xLength = (int)(this.xLength * decay);
        this.yLength = (int)(this.yLength * decay);
        this.x = this.ifXPositive ? this.x + this.xLength : this.x - this.xLength;
        this.y = this.ifYPositive ? this.y + this.yLength : this.y - this.yLength;
        if (this.x <= padding) {
            this.x = padding;
            this.ifXPositive = !this.ifXPositive;
        }
        if (this.x >= this.getWidth() - padding - diameter) {
            this.x = this.getWidth() - padding - diameter;
            this.ifXPositive = !this.ifXPositive;
        }
        if (this.y <= titleWidth) {
            this.y = titleWidth;
            this.ifYPositive = !this.ifYPositive;
        }
        if (this.y >= this.getHeight() - padding - diameter) {
            this.y = this.getHeight() - padding - diameter;
            this.ifYPositive = !this.ifYPositive;
        }
    }

    private class RepaintRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    GameFrame.this.repaint();
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

    public static void main(String[] args) {
        new GameFrame().launchFrame();
    }
}
```