---
title: Java IO-File类
date: 2017-12-09 12:02:49
tags: [Java,IO,流,File]
categories: Java IO
---

java.io.File类以文件的全路径名为核心要素，操作文件的各项属性。换句话说，一个File类的实例就代表一个文件。这里需要注意的是在File类眼中我们通常意义上的文件和目录并没有本质上的区别，在大的类别上都会被其视为文件。

<!-- more -->

# 统一的分隔符

以Windows系统为例，E盘下的a目录下的b.txt文件的全路径名可表示为：

```
E:\a\b.txt
```

但是我们却无法直接在Java中定义这样的字符串：

```
String filePath = "E:\a\b.txt";
```

会无法通过编译，Eclipse中的提示为：

```
Invalid escape sequence (valid ones are \b \t \n \f \r \" \' \\ )
```

很显然，'\'这个字符是转义字符的前置标记，像上例那么写会引起编译器的迷惑，它不知道我们是真的想输出'\'还是想将'\'后的字符转义。因此，如果我们真的想输出'\'，只能这样写：

```
String filePath = "E:\\a\\b.txt";
```

从书写的角度上来讲，这是不符合人类的认知习惯的。

不仅如此，以上仅仅只是Windows系统的路径规范，到了其他操作系统中一般都会有所不同。例如Linux系统中类似文件的全路径为:

```
e/a/b.txt
```

此时就不涉及转义问题而可直接在Java中声明：

```
String filePath = "e/a/b.txt";
```

这其实是相悖于Java的平台无关性口号的，因为我们此时已经需要根据底层操作系统的不同编写不同的代码了。同时还牵扯到让人难以理解的转义问题。

为了最大限度的解决这个问题，File类中提供了读取系统目录连接字符的类变量：

```
public static final char separatorChar = fs.getSeparator();
```

这个字段读出来的就是当前系统的连接符了。例如Windows下是'\'，而Linux下是'/'。并且也不会涉及到转义的问题。

考虑到比起char，大家应该更习惯使用String，因此File类中还贴心的提供了String版本的连接符：

```
public static final String separator = "" + separatorChar;
```

这样在我的Windows系统中的下述代码：

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder("E:").append(File.separator)
                                                  .append("a")
                                                  .append(File.separator)
                                                  .append("b.txt");
        System.out.println(sb.toString());
    }
}
```

输出：

```
E:\a\b.txt
```

最后需要说明的是，虽然Windows系统下默认的连接符是'\'，然而Java却也没那么死板。写成'/'在Windows系统中其实也能识别：

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        File file = new File("E:/a/b.txt");
        System.out.println(file.getPath());
    }
}
```

输出：

```
E:\a\b.txt
```

很显然，输出结果又变回了Windows系统默认的'\'，显然是其内部有所转换。当然这个灵活性也要有个限度，例如Windows系统下如果写成这个样子：

```
File file = new File("E/a/b.txt");
```

再或者：

```
File file = new File("/E/a/b.txt");
```

再或者：

```
File file = new File("E\\a\\b.txt");
```

再或者：

```
File file = new File("\\E\\a\\b.txt");
```

总之，要是连盘符划分都去掉，即不写":"，那只靠Java API核心类库是肯定读不出文件的。

因此个人建议干脆放弃这个其实不那么灵活的灵活性，既然File类中已经给出了类变量，那么就尽量不要写硬编码。退一步讲，即便写了硬编码，也最好用系统默认的连接符。例如如果要使用Windows系统的文件系统，那么连接符就用'\'，而不是'/'。

# 构造函数

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("a")
                                             .toString();
        File file = new File(path);
        System.out.println(file.getPath());
        System.out.println(file.exists());
    }
}
```

输出：

```
D:\a
false
```

直接以全路径名构建File对象。这是最直接也是最常用的构造方式。注意构建File对象并不需要被构建的文件一定真实存在，例如本例中就是这样。

--- 

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path0 = new StringBuilder("D:").append(File.separator)
                                              .append("a")
                                              .toString();
        File file0 = new File(path0);
        File file1 = new File(file0, "b.txt");
        System.out.println(file1.getParent());
        System.out.println(file1.getPath());
    }
}
```

输出：

```
D:\a
D:\a\b.txt
```

因为目录结构总是有所属关系的，因此在已有File对象内部(此时该File对象必定是个目录)创建新File对象也是常见的需求。这比直接使用全路径名创建File对象更符合层级关系，某些情况下更易于被开发人员所理解。例如上例中，我们就以file0为父目录，创建了新文件file1。

当然，上文所说的"该File对象必定是个目录"在构造File对象时依然没有检查(连是否存在都不检查又怎么会检查到底是文件还是目录呢？)。由此可见，File类的检查限制是相对宽松的：磁盘中实际存在的文件是一回事，内存中声明的File对象又是另一回事。为什么这么做的原因也是很好理解的：Java怎么会知道你新建一个File对象的目的是什么呢？也许你就是想在磁盘上新创建一个本不存在的文件呢？此时没有才是正常的。因此除非具体操作产生矛盾(例如明明磁盘上没有这个文件却当是有那样读里面的内容，再或者明明实际是个文件却当是目录那样在内部创建新文件等等)，否则不会在创建File对象时就进行严格的检查。

如果父目录还没来得及生成File对象，也可按如下方式来做：

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path0 = new StringBuilder("D:").append(File.separator)
                                              .append("a")
                                              .toString();
        File file1 = new File(path0, "b.txt");
        System.out.println(file1.getParent());
        System.out.println(file1.getPath());
    }
}
```

输出：

```
D:\a
D:\a\b.txt
```

显然，效果是一样的。

---

我们还可以以URI对象为核心构造File类对象：

```
import java.io.File;
import java.net.URI;
import java.net.URISyntaxException;

public class Test {
    public static void main(String[] args) throws URISyntaxException {
        URI uri = new URI("file:/D:/a");
        File file = new File(uri);
        System.out.println(file.getPath());
	System.out.println(file.exists());
    }
}
```

输出：

```
D:\a
false
```

URI(Uniform Resource Identifier)，即统一资源标识符，一如其名，即试图用一个统一的格式描述来源不同的资源文件。正如上例中提到的，存储于文件系统中的URI字符串为：

```
file:/D:/a
```

依据URI，我们同样可以构建出对应的File类对象。显然构建URI类对象时仍然不会检查资源是否存在。

关于URI，还需注意的一点是，它已经是超脱于实际操作系统之上的抽象概念了。因此其资源路径的表示方法是固定的，和具体的操作系统无关。上例中，不管使用什么文件系统，都是(当然，Linux下D:中的盘符:要去掉)：

```
URI uri = new URI("file:/D:/a");
```

即便在Windows系统中，使用

```
URI uri = new URI("file:\\D:\\a");
```

也是错误的。

# File对象是否真的存在

假设：D盘下有目录a，a中有文件b.txt。

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path0 = new StringBuilder("D:").append(File.separator)
                                              .append("a")
                                              .toString();
        File file0 = new File(path0);
        File file1 = new File(file0, "b.txt");
        System.out.println(file0.exists());
        System.out.println(file1.exists());

    }
}
```

输出：

```
true
true
```

# 实际是目录还是文件

假设：D盘下有目录a，a中有文件b.txt。

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path0 = new StringBuilder("D:").append(File.separator)
                                              .append("a")
                                              .toString();
        File file0 = new File(path0);
        File file1 = new File(file0, "b.txt");
        System.out.println(file0.isDirectory());
        System.out.println(file0.isFile());
        System.out.println(file1.isDirectory());
        System.out.println(file1.isFile());
    }
}
```

输出：

```
true
false
false
true
```

# 访问权限

假设：D盘下有目录a，a中有文件b.txt。

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path0 = new StringBuilder("D:").append(File.separator)
                                              .append("a")
                                              .toString();
        File file0 = new File(path0);
        File file1 = new File(file0, "b.txt");
        System.out.println(file0.isHidden());
        System.out.println(file0.canRead());
        System.out.println(file0.canWrite());
        System.out.println(file0.canExecute());
        System.out.println(file1.isHidden());
        System.out.println(file1.canRead());
        System.out.println(file1.canWrite());
        System.out.println(file1.canExecute());
    }
}
```

输出：

```
false
true
true
true
false
true
true
true
```

需要注意的是，a作为一个目录也被判为可执行了。显然这些权限判断方法仅仅关心是否有权限，至于被操作的对象能不能做到则不在乎。这又是和整个File类一脉相承的设计思路一致。

# 最后修改时间

假设：D盘下有目录a

```
import java.io.File;
import java.util.Date;

public class Test {
    public static void main(String[] args) {
        String path = new StringBuilder("D:").append(File.separator)
                                              .append("a")
                                              .toString();
        File file = new File(path);
        long time = file.lastModified();
        System.out.println(new Date(time));
    }
}
```

输出：

```
Sun Dec 10 21:30:33 CST 2017
```

# 基本信息

假设：D盘下有目录a，a中有文件b.txt。

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path0 = new StringBuilder("D:").append(File.separator)
                                              .append("a")
                                              .toString();
        File file0 = new File(path0);
        File file1 = new File(file0, "b.txt");
        System.out.println(file0.getName());    // 简单文件名
        System.out.println(file0.getPath());    // 全路径名
        System.out.println(file0.length());    // 长度，单位为byte(目录的长度默认为0)
        System.out.println("=======================");
        System.out.println(file1.getName());    // 简单文件名
        System.out.println(file1.getPath());    // 全路径名
        System.out.println(file1.length());    // 长度，单位为byte
    }
}
```

输出：

```
a
D:\a
0
=======================
b.txt
D:\a\b.txt
11405
```

# 父文件

所谓父文件，若某文件在某目录下，那么该目录就相当于该文件的父文件。假设：D盘下有目录a，a中有文件b.txt。此时a是b.txt的父文件。D:是a的父文件。显然只有目录才有资格做父文件：

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path0 = new StringBuilder("D:").append(File.separator)
                                              .append("a")
                                              .toString();
        File file0 = new File(path0);
        File file1 = new File(file0, "b.txt");
        System.out.println(file0.getParent());    // 获得父文件全路径名
        File file0p = file0.getParentFile();    // 获得父文件实例
        System.out.println(file0p.getPath());
        System.out.println("=======================");
        System.out.println(file1.getParent());
        File file1p = file1.getParentFile();
        System.out.println(file1p.getPath());
    }
}
```

输出：

```
D:\
D:\
=======================
D:\a
D:\a
```

那么我们不禁好奇，作为顶层目录的D盘的父文件会是什么呢？

```
import java.io.File;

public class Test {
    public static void main(String[] args) {
        String path = new StringBuilder("D:").append(File.separator)
                                              .toString();
        File file = new File(path);
        System.out.println(file.getParent());
        System.out.println(file.getParentFile());
    }
}
```

输出：

```
null
null
```

输出都是null，合情合理。

# 获得目录下所有内容

```
import java.io.File;

public class Test {

    public static void main(String[] args) {
        String path = new StringBuilder("D:").append(File.separator)
                                             .toString();
        File root = new File(path);
        for(File file : root.listFiles()) {
            String type = file.isFile() ? "文件" : "文件夹";
            String hiden = file.isHidden() ? "(隐藏)" : "(可见)";
            System.out.println(type + hiden + "\t" + file.getName());
        }
    }
}
```

输出：

```
文件夹(隐藏)	$RECYCLE.BIN
文件夹(隐藏)	15055
文件夹(隐藏)	360Downloads
文件夹(隐藏)	360Rec
文件夹(隐藏)	Driver
文件夹(可见)	game
文件夹(隐藏)	MSOCache
文件夹(可见)	program
文件夹(隐藏)	SoftwareDistribution
文件夹(隐藏)	System Volume Information
文件夹(可见)	temp
文件(可见)	test.txt
文件夹(隐藏)	WindowsApps
文件夹(可见)	work
文件夹(隐藏)	WpSystem
文件夹(隐藏)	WUDownloadCache
```

# 实际创建文件

```
import java.io.File;
import java.io.IOException;

public class Test {

    public static void main(String[] args) throws IOException {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("Reimu.txt")
                                             .toString();
        File file = new File(path);
        System.out.println(file.createNewFile());
    }
}
```

在没有D:\Reimu.txt的前提下，上述代码执行后，D:\Reimu.txt会被创建。程序输出true。

记录下此时的文件属性：

![0.jpg](/images/blog_pic/Java IO/File类/0.jpg)

那么我们不禁会想：如果创建前文件已存在会怎么样呢？于是再次执行代码。程序输出false。再看D:\Reimu.txt，发现没有任何变化。

难道说，如果待创建文件已存在，便输出一个false，然后什么都不做吗？

为了验证上述猜想，我们不妨手动修改D:\Reimu.txt的内容，比如在其中添加文本：魔理沙么么哒。然后此时我们再看文件属性：

![1.jpg](/images/blog_pic/Java IO/File类/1.jpg)

理所当然的，文件大小及修改时间发生了相应的变化。此时我们再执行上述代码，依然输出false。随后我们再观察D:\Reimu.txt的属性，发现没有任何变化，内容也依然是魔理沙么么哒(若代码生效了内容应被替换为空文件)。说明猜想成立。

# 实际创建目录

```
import java.io.File;
import java.io.IOException;

public class Test {

    public static void main(String[] args) throws IOException {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("Reimu.txt")
                                             .toString();
        File file = new File(path);
        System.out.println(file.mkdir());
    }
}
```

在没有D:\Reimu.txt存在的情况下，上述代码输出true。并成功创建了目录D:\Reimu.txt。注意这里创建的Reimu.txt看似是txt文件，然而实则是目录。我之所以起这么诡异的名字，就是为了说明Java之所以统一使用File类描述文件和目录，在底层没有对二者做本质上的区分是有道理的：因为确实也没那么大的差别，起码无法从名字上就能确认到底是文件还是目录。

同理，我们再执行一次上述代码，输出false。说明创建失败，什么都不做。

需要注意的是，若D盘下存在文件Reimu.txt(注意，这次真的是文件了，不是目录)，此时执行上述代码依然输出false。说明即便一个是目录，一个是文件，依然不允许重名。事实上，如果在Windows系统下通过图形化界面直接这么做，依然不会通过：

![2.jpg](/images/blog_pic/Java IO/File类/2.jpg)

看来不仅是Java，Windows系统也没有对文件和目录做本质上的区分。

然后我们再看以下代码：

```
import java.io.File;
import java.io.IOException;

public class Test {

    public static void main(String[] args) throws IOException {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("dir1")
                                             .append(File.separator)
                                             .append("dir2")
                                             .toString();
        File file = new File(path);
        System.out.println(file.mkdir());
    }
}
```

目录dir1及目录dir2均不存在，此时执行上述代码输出false。D盘下也未创建任何目录。这也很合理：dir1都不存在，自然无法创建其里层的dir2。但同时这又是一个很常见的需求，就是要直接连续创建目录，难道此时只能将创建过程断成很多截，一层目录一层目录的创建吗？

为了解决这个问题，File类提供了如下解决方案：

```
import java.io.File;
import java.io.IOException;

public class Test {

    public static void main(String[] args) throws IOException {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("dir1")
                                             .append(File.separator)
                                             .append("dir2")
                                             .toString();
        File file = new File(path);
        System.out.println(file.mkdirs());
    }
}
```

目录dir1及目录dir2均不存在，此时执行上述代码输出true。同时按目录层级D:\dir1\dir2创建了目录dir1及目录dir2。

# 实际删除文件/目录

```
import java.io.File;
import java.io.IOException;

public class Test {

    public static void main(String[] args) throws IOException {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("1.txt")
                                             .toString();
        File file = new File(path);
        System.out.println(file.delete());
    }
}
```

已有文件D:\1.txt的前提下，上述代码执行后输出true。同时磁盘中的文件被删除。需要注意的是，这种删除方式非常霸道：被删除的文件是彻底被删除了，并非被放到回收站等缓冲地带中。

当然，若待删除文件实际不存在，则输出false，并什么都不做。

关于目录的删除方式大同小异：

```
import java.io.File;
import java.io.IOException;

public class Test {

    public static void main(String[] args) throws IOException {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("dir1")
                                             .toString();
        File file = new File(path);
        System.out.println(file.delete());
    }
}
```

D盘下存在目录dir1，dir1中为空。上述代码执行后返回true。同时D盘下的dir1被删除，且依然是彻底删除。

不过若dir1下不为空，比如说有目录dir2或文件1.txt，那么上述代码执行后返回false，并且什么都不会做，即只要目录下有内容就不会删除目录。这样设计是很有必要的，因为正如我们前文所述，Java的这种删除方法实在是过于霸道且毫无容错机制，那么如果我们一个手抖不小心将删除的目录写成了D:\，如果自动递归删除目录下的所有内容的话，那相当于直接把D盘给格式化了。当然说格式化其实有些过了，因为毕竟有些文件Java是没有操作权限的，不过即便如此，也依然是够糟糕的了。

# 小例子：递归打印目录层级

```
import java.io.File;

public class Test {

    private static void printFileSystemRecursion(File file, int depth) {
        boolean isDirectory = file.isDirectory();
        StringBuilder sb = new StringBuilder("[");
        sb = isDirectory ? sb.append("d") : sb.append("-");
        sb = file.canRead() ? sb.append("r") : sb.append("-");
        sb = file.canWrite() ? sb.append("w") : sb.append("-");
        sb = file.canExecute() ? sb.append("x") : sb.append("-");
        sb = file.isHidden() ? sb.append(".") : sb.append("-");
        sb.append("]").append(file.getName());
        for (int i = 0; i < depth; i++) System.out.print("       ");
        System.out.println(sb.toString());
        if (!isDirectory) return;
        for(File temp : file.listFiles()) Test.printFileSystemRecursion(temp, depth + 1);
    }

    private static void printFileSystem(File file) {
        if (!file.exists()) {
            System.out.println(file.getPath() + "不存在");
            return;
        }
        Test.printFileSystemRecursion(file, 0);
    }

    public static void main(String[] args) {
        String path = new StringBuilder("D:").append(File.separator)
                                             .append("work")
                                             .append(File.separator)
                                             .append("java")
                                             .append(File.separator)
                                             .append("javaSoft")
                                             .append(File.separator)
                                             .append("jdk")
                                             .append(File.separator)
                                             .append("include")
                                             .toString();
        File file = new File(path);
        Test.printFileSystem(file);
    }
}
```

输出：

```
[drwx-]include
       [-rwx-]classfile_constants.h
       [-rwx-]jawt.h
       [-rwx-]jdwpTransport.h
       [-rwx-]jni.h
       [-rwx-]jvmti.h
       [-rwx-]jvmticmlr.h
       [drwx-]win32
              [drwx-]bridge
                     [-rwx-]AccessBridgeCallbacks.h
                     [-rwx-]AccessBridgeCalls.c
                     [-rwx-]AccessBridgeCalls.h
                     [-rwx-]AccessBridgePackages.h
              [-rwx-]jawt_md.h
              [-rwx-]jni_md.h
```