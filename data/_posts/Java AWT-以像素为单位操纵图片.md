---
title: Java AWT-以像素为单位操纵图片
date: 2018-05-20 16:54:55
tags: [Java,GUI,AWT,图片]
categories: Java AWT
---

我们先准备一张示例图片，将其命名为0.jpg并放在D盘根目录下：

![0.jpg](/images/blog_pic/Java AWT/以像素为单位操纵图片/0.jpg)

<!-- more -->

通过Windows属性查看器，可以看到该图片的属性信息中有如下描述：

![1.jpg](/images/blog_pic/Java AWT/以像素为单位操纵图片/1.jpg)

这也就是咱们常说的图片的宽与高。那么从本质上讲，它们到底是什么呢？

简单来说，我们看到的绝大多数图片都是RGB图片。而从本质上讲，RGB图片是一个二维数组。数组中的元素是RGB像素点(Red-Green-Blue三基色调和而成)。也就是说，图片本质上是离散的，但是如果像素点足够密集的话，便可以欺骗人的眼睛，让其看起来是连续的。

回到本例，本例中的图片其实就是一个这样的二维数组：数组有711行，每行有1280个像素点。

那么，如果我们想操作图片，最底层，也是灵活性最高的做法自然就是以像素点为单位了，这样，我们就可以很容易的做到任何我们能想到的和图片相关的操作了。

下面给出一个小例子：

```
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;

public class Test {

    public static void main(String[] args) throws IOException {
        String oldPath = new StringBuilder("D:").append(File.separator)
                                                .append("0.jpg")
                                                .toString();
        File oldFile = new File(oldPath);
        BufferedImage bufferedImage = ImageIO.read(oldFile);
        String newPath = new StringBuilder("D:").append(File.separator)
                                             .append("1.jpg")
                                             .toString();
        File newFile = new File(newPath);
        ImageIO.write(bufferedImage, "jpg", newFile);
    }
}
```

执行该代码后，D盘下将生成1.jpg，因为我们将内存中的bufferedImage原样输出了，因此它与原有的0.jpg一模一样。

下面我们来看下bufferedImage中的像素点：

```
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;

public class Test {

    public static void main(String[] args) throws IOException {
        String oldPath = new StringBuilder("D:").append(File.separator)
                                                .append("0.jpg")
                                                .toString();
        File oldFile = new File(oldPath);
        BufferedImage bufferedImage = ImageIO.read(oldFile);
        for (int y = 0; y < bufferedImage.getHeight(); y++) {
            for (int x = 0; x < bufferedImage.getWidth(); x++) {
                int p = bufferedImage.getRGB(x, y);
                int r = (p & 0xff);
                int g = (p & 0xff00) >> 8;
                int b = (p & 0xff0000) >> 16;
                System.out.println("[R,G,B]=[" + r + "," + g + "," + b + "]");
            }
        }
        String newPath = new StringBuilder("D:").append(File.separator)
                                             .append("1.jpg")
                                             .toString();
        File newFile = new File(newPath);
        ImageIO.write(bufferedImage, "jpg", newFile);
    }
}
```

图片输出依然没变，不过这次我们还逐行打印了该图片所有像素点的RGB值，它的部分输出如下：

```
[R,G,B]=[172,148,136]
[R,G,B]=[172,149,134]
[R,G,B]=[174,147,133]
[R,G,B]=[171,144,130]
[R,G,B]=[166,140,126]
[R,G,B]=[162,136,122]
[R,G,B]=[155,131,119]
[R,G,B]=[147,125,114]
[R,G,B]=[139,115,109]
[R,G,B]=[133,108,104]
[R,G,B]=[121,100,99]
```

接着我们需要稍微补充一些RGB的基础知识了。

常见的图片中的RGB都是8位真彩色。也就是分别用8位来表示一种单色，因为2^8=256，即有256种红色，256种绿色，256种蓝色。这三种基色再进行调和，共可以得到2^24种颜色，这其实已经是一个足够大的数字了，临近的两种离散的颜色的色差其实已经很小了，用来欺骗我们的眼睛基本是足够了。正如上文的输出结果所证实的，RGB像素点每一个分量的值均在[0,255]的范围内。

然后我们再简要介绍下Java中RGB像素点的存储方式。Java是使用一个整型数据来存储一个RGB像素点的。1个整型数据长32位，RGB像素点只使用最后的24位。在这24位中：首先是8位的B，然后是8位的G，最后是8位的R。这也是上文代码的取值依据。

知道了这些，再要操作图片就容易了：

```
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import javax.imageio.ImageIO;
public class Test {
    public static void main(String[] args) throws IOException {
        String oldPath = new StringBuilder("D:").append(File.separator)
                                                .append("0.jpg")
                                                .toString();
        File oldFile = new File(oldPath);
        BufferedImage bufferedImage = ImageIO.read(oldFile);
        for (int y = 0; y < bufferedImage.getHeight(); y++) {
            for (int x = 0; x < bufferedImage.getWidth(); x++) {
                int p = bufferedImage.getRGB(x, y);
                int r = p & 0xff;
                int g = (p & 0xff00) >> 8;
                int b = (p & 0xff0000) >> 16;
                int newP = (255 - r) | ((255 - g) << 8) | ((255 - b) << 16);
                bufferedImage.setRGB(x, y, newP);
            }
        }
        String newPath = new StringBuilder("D:").append(File.separator)
                                             .append("1.jpg")
                                             .toString();
        File newFile = new File(newPath);
        ImageIO.write(bufferedImage, "jpg", newFile);
    }
}
```

例如，上述代码将1.jpg中的每个像素点都置为了0.jpg对应位置像素点的反色，即：

![2.jpg](/images/blog_pic/Java AWT/以像素为单位操纵图片/2.jpg)

再比如：

```
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;

public class Test {

    public static void main(String[] args) throws IOException {
        String oldPath = new StringBuilder("D:").append(File.separator)
                                                .append("0.jpg")
                                                .toString();
        File oldFile = new File(oldPath);
        BufferedImage bufferedImage = ImageIO.read(oldFile);
        final int width = 2;
        boolean ifShowY = true;
        int nowY = 0;
        for (int y = 0; y < bufferedImage.getHeight(); y++) {
            if (!ifShowY) {
                for (int x = 0; x < bufferedImage.getWidth(); x++)
                    bufferedImage.setRGB(x, y, 0);
            } else {
                boolean ifShowX = true;
                int nowX = 0;
                for (int x = 0; x < bufferedImage.getWidth(); x++) {
                    if(!ifShowX) bufferedImage.setRGB(x, y, 0);
                    nowX++;
                    if (nowX == width) {
                        nowX = 0;
                        ifShowX = !ifShowX;
                    }
                }
            }
            nowY++;
            if (nowY == width) {
                nowY = 0;
                ifShowY = !ifShowY;
            }
        }
        String newPath = new StringBuilder("D:").append(File.separator)
                                             .append("1.jpg")
                                             .toString();
        File newFile = new File(newPath);
        ImageIO.write(bufferedImage, "jpg", newFile);
    }
}
```

上述代码实现了一种网格化效果，每个小网格的大小位2*2。因为有3/4的点没有输出，因此图片显得很暗：

![3.jpg](/images/blog_pic/Java AWT/以像素为单位操纵图片/3.jpg)