---
title: JVM-类加载器
date: 2017-12-04 14:15:49
tags: [Java,JVM,Java虚拟机,虚拟机,类加载器]
categories: JVM
---

[JVM-类加载机制](/2017/11/23/JVM-类加载机制/)中已讨论过了，JVM规范对于类加载-加载阶段中的限制很少，或者更具体的来说，是对这一阶段中"通过类的全限定名获取该类的二进制字节流"这一动作的限制很少，这为JVM的实现者提供了很高的灵活性。其原理为该动作的绝大多数的实现逻辑被放到了JVM之外(最核心的那一组类的加载逻辑依然控制在JVM内部)，从而其才有可能会被用户代码所影响。实现这一动作的代码模块被称为"类加载器"。

该机制诞生的初衷是为了实现Java Applet。Java最初的崛起依托的也是这种小程序。只是，随着SUN与微软那场著名的世纪撕逼大战的升级，Java Applet技术逐渐式微，至今基本上已完全死掉了(最初的主力业务，浏览器上的Java Applet算是彻底完蛋了。在其他领域，例如智能卡，Java Applet还有一定的生存空间)。然而失之东隅，收之桑榆，类加载器技术本身却探索出了新的发展道路，现在其已是Java技术体系中重要的基石，在类层次划分，OSGi，热部署，代码加密等领域大放异彩。

<!-- more -->

除了完成"通过类的全限定名获取该类的二进制字节流"这一具体的动作之外，类加载器还起到了重要的类身份识别作用。如果我们将类比作Mybatis配置文件中的statement，那么类加载器就是statement所属的namespace(每一个类加载器都有一个独立的类名称空间)。正如namespace+statement id唯一标识一个statement那样，类加载器+类则唯一标识一个类。更通俗的来说，比较内存中的两个类是否是同一个，必要的前提条件是二者的类加载器相同。否则，即便这两个类来源于同一个class文件，被加载入同一个JVM实现，只要加载它们的类加载器不同，它们就必定不相等。

具体到Java代码层面，这里的"不相等"会体现在类的Class对象的equals()方法，isAssignableFrom()方法，isInstance()方法等方法的返回结果上。同时既然影响了类对象的isInstance()方法，那么自然也会影响实例的instanceof比较。

为了证明这一点，我们自行设计一个类构造器：

```
package com.test;

import java.io.IOException;
import java.io.InputStream;

public class Test { 

    public static void main(String[] args) throws Exception {
        ClassLoader myClassLoader = new ClassLoader() {
            @Override
            public Class<?> findClass(String name) throws ClassNotFoundException {
                try{
                    String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                    InputStream is = this.getClass().getResourceAsStream(fileName);
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length );
                } catch(IOException e){
                    throw new ClassNotFoundException(name);
                }
            }
        };
        Object obj = myClassLoader.loadClass("com.test.Test").newInstance();
        System.out.println(obj.getClass());
        System.out.println(obj instanceof com.test.Test);
    }
}
```

输出：

```
class com.test.Test
true
```

这是Java API中推荐的类加载器的自定义方式，即继承ClassLoader类并重写其内部的findClass方法。不过结果却很遗憾的并不符合我们的预期：比较结果是true。难道是前文介绍的类加载器机制有误？还是obj并非是由我们自定义的myClassLoader加载的？

答案是后者。Java默认并且推荐开发人员遵守其所构造的双亲委派模型(将在后文详述)，简单来说，就是当我们试图用一个类加载器加载某一个类前，会先尝试用该类加载器的父类加载器(这里说的父类是逻辑上的继承关系，并非一定就是写在代码里的那种继承)加载，若父类加载器加载不了，再交给本来打算使用的类加载器。显然，这是一个递归动作，任何一个类加载器的加载操作都会追溯到最顶层的那个类加载器，而后逐级向下查找。直到找到能够加载目标类的类加载器。在上例中，我们所定义的myClassLoader的父类加载器：应用程序类加载器已有能力加载目标类，自然轮不到myClassLoader出手了。

不过有时因为某些特定需求，即便Java不推荐，我们依然不得不破坏双亲委派模型。比如我们现在面临的就是这样的一个"特定的需求"：我们想看看不同类加载器加载到内存里的类到底会不会被判为不相等。为此，我们只能采用Java不推荐的方法，即一步到位，直接重写ClassLoader类中最终用于加载的loadClass方法，并在其内部指定我们自己破坏双亲委派模型的逻辑：无视父类加载器，只用当前我们自定义的类加载器加载：

```
package com.test;

import java.io.IOException;
import java.io.InputStream;

public class Test { 

    public static void main(String[] args) throws Exception {
        ClassLoader myClassLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                try {
                    String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                    InputStream is = this.getClass().getResourceAsStream(fileName);
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length);    // 17行
                } catch (IOException e) {
                    throw new ClassNotFoundException(name);
                }
            }
        };
        Object obj = myClassLoader.loadClass("com.test.Test").newInstance();
        System.out.println(obj.getClass());
        System.out.println(obj instanceof com.test.Test);
    }
}
```

输出：

```
Exception in thread "main" java.lang.NullPointerException
	at com.test.Test$1.loadClass(Test.java:15)
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:800)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:643)
	at com.test.Test$1.loadClass(Test.java:17)
	at com.test.Test.main(Test.java:23)
```

结果依然很糟心(人生不如意事十之八九=-=)，报错的地方是17行。我进到了这个defineClass方法内部，一路跟踪。发现它会以目标类的父类为参数再次调用我们自定义的类加载器。由于这是个递归的过程，看来是会一直追溯到Object类了。于是我尝试输出了一下fileName：

```
package com.test;

import java.io.IOException;
import java.io.InputStream;

public class Test { 

    public static void main(String[] args) throws Exception {
        ClassLoader myClassLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                try {
                    String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                    System.out.println(fileName);
                    InputStream is = this.getClass().getResourceAsStream(fileName);
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length);
                } catch (IOException e) {
                    throw new ClassNotFoundException(name);
                }
            }
        };
        Object obj = myClassLoader.loadClass("com.test.Test").newInstance();
        System.out.println(obj.getClass());
        System.out.println(obj instanceof com.test.Test);
    }
}
```

输出：

```
Test.class
Object.class
Exception in thread "main" java.lang.NullPointerException
	at com.test.Test$1.loadClass(Test.java:16)
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:800)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:643)
	at com.test.Test$1.loadClass(Test.java:18)
	at com.test.Test.main(Test.java:24)
```

果然，我们的loadClass被调用了两次，而第二次传入的java.lang.Object类我们自定义的类加载器当然没有加载的权限，is自然是null。知道了症结所在，解决起来就简单了。加载不了继续委派给父类加载器就可以了。看来父类加载器并非完全没用，我们的破坏实际上相当于将双亲委派模型的查询顺序倒了过来：

```
package com.test;

import java.io.IOException;
import java.io.InputStream;

public class Test { 

    public static void main(String[] args) throws Exception {
        ClassLoader myClassLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                try {
                    String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                    InputStream is = this.getClass().getResourceAsStream(fileName);
                    if (null == is) return super.loadClass(name);
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length);
                } catch (IOException e) {
                    throw new ClassNotFoundException(name);
                }
            }
        };
        Object obj = myClassLoader.loadClass("com.test.Test").newInstance();
        System.out.println(obj.getClass());
        System.out.println(obj instanceof com.test.Test);
    }
}
```

输出：

```
class com.test.Test
false
```

终于我们成功了，也顺利验证了前文理论知识的正确性。

# 双亲委派模型

从JVM的角度来看只有两种类加载器，也就是被留在自身内部的"核心"及被丢到自身之外的"弃儿"：

- 启动类加载器(Bootstrap ClassLoader)：即被留在JVM内部的"核心"。其使用C++实现(强调一下，特指HotSpot VM)。
- 其他类加载器。即独立于JVM外部的"弃儿"。均采用Java语言实现，并均继承自抽象类java.lang.ClassLoader。

启动类加载器负责加载如下核心类：

- &lt;JAVA_HOME&gt;/lib目录(这里的&lt;JAVA_HOME&gt;指的是jre)下的类。
- 被-Xbootclasspath参数所指定的路径中的类。

除此之外，还需要类所属的文件名能被JVM所识别。换句话说，启动类加载器的类加载检查其实很严格，只能加载有数的那么几个系统核心类(例如rt.jar)，并不是随便将个jar包丢到&lt;JAVA_HOME&gt;/lib目录下启动类加载器就会去加载。

既然启动类加载器被保留在了JVM内部，那么很显然，开发人员无法直接引用它。对于HotSpot而言，被启动类加载器加载的类的类加载器是null，仿佛它不存在一样：

```
package com.test;

public class Test {

    public static void main(String[] args) {
        System.out.println(Object.class.getClassLoader());
        System.out.println(Test.class.getClassLoader());
    }
}
```

输出：

```
null
sun.misc.Launcher$AppClassLoader@e1641c0
```

显然，除了启动类加载器之外，JVM对于它的弃儿们过于的冷漠了，一句"其他"就全打发了。这些"其他"其实又可分为两类：

- 扩展类加载器(Extension ClassLoader)
- 应用程序类加载器(Application ClassLoader)

扩展类加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载如下类：

- &lt;JAVA_HOME&gt;/lib/ext目录(这里的&lt;JAVA_HOME&gt;指的是jre)下的类。
- 被系统变量java.ext.dirs所指定的路径中的类。

从名称上就可以看出来，扩展类加载器加载的是核心类库的扩展，自然也就没那么重要了。因此才会被移出JVM，用户代码也可以直接使用该类加载器。

---

应用程序类加载器由sun.misc.Launcher$AppClassLoader实现。很显然，较之扩展类加载器，应用程序类加载器更加的远离核心，它负责加载用户路径(也就是我们在安装Java时通常都会配置的那个环境变量ClassPath)中指定的类库，用户代码自然也可以直接使用该类加载器。因此，我们日常写的代码如果没有做特别的关于类加载器方面的操作(例如上文我们强行破坏双亲委派模型的例子)，默认的类加载器就是这个应用程序类加载器。

```
public class Test {

    public static void main(String[] args) {
        System.out.println(ClassLoader.getSystemClassLoader());
    }
}
```

输出：

```
sun.misc.Launcher$AppClassLoader@7d05e560
```

如上代码所示，ClassLoader的类方法getSystemClassLoader默认返回的类加载器就是这个应用程序类加载器。因此它也被称作系统类加载器。从这里我们也能看出，对于Java设计者而言，他们是希望用户能尽量不去干扰更上一层的扩展类加载器(然后完全忘掉启动类加载器)，而以应用程序类加载器为代码中类加载器的根。

---

至此，Java预定义的类加载器已全部介绍完毕。如果有必要，用户可以再写自定义的用户类加载器(User ClassLoader)。其结构关系通常如下图所示：

![0.png](/images/blog_pic/JVM/类加载器/0.png)

这种层次继承关系被称为类加载器的双亲委派模型(Parents Delegation Model)。需要注意的是，这里的继承并不是指Java语法上的继承，而是一种更抽象的，表达执行次序的关系。例如，前文我们所做的破坏双亲委派模型，我们的匿名内部类直接继承的是抽象类ClassLoader，然而在双亲委派模型的层次继承关系中，我们自定义的这个用户类加载器依然是应用程序类加载器的孩子。或者我们可以用更准确的词语来表达：双亲委派模型中类加载器间的父子关系并非继承(Inheritance)，而是以组合(Compositon)的方式复用。父亲更像是儿子执行的前置条件。

所谓双亲委派模型，就是当我们试图用一个类加载器加载某一个类前，会先尝试用该类加载器的父类加载器加载，若父类加载器加载不了(它的搜索范围中并没有目标类)，再交给本来打算使用的类加载器。显然，这是一个递归动作，任何一个类加载器的加载操作都会追溯到最顶层的启动类加载器，而后逐级向下查找。直到找到能够加载目标类的类加载器。

双亲委派模型诞生自JDK1.2，所有类默认遵守该模型。然而，双亲委派模型并非是强制规范(例如，上文我们就写出了破坏双亲委派模型的代码)，而只是Java设计者的一种推荐。之所以如此推荐，就是为了保证类的唯一性，否则在内存中可能就会产生大量仅仅是因为类加载器不同而导致的不同的类。例如&lt;JAVA_HOME&gt;/lib目录下rt.jar中的java.lang.Object类，由于双亲委派模型的存在，内存中只会存在一个java.lang.Object类，且该类必然由启动类加载器加载。

即便某些开发人员写出了自己的java.lang.Object类，并放在ClassPath下，这个自定义的类依然无法被加载。因为按照双亲委派模型，该类的加载请求会先被送到顶层的启动类加载器中。启动类加载器一检查，java.lang.Object类已经在此前被自己加载过了(就是rt.jar中那个真正的Object)，因此直接就把这个加载结果返回了。自定义的java.lang.Object类则永无加载机会。

上文中我们既写出了破坏双亲委派模型的用户类加载器。也写出了顺应双亲委派模型，官方推荐的通过重写findClass方法实现的用户类加载器。 那么为什么重写了findClass方法双亲委派模型就能生效呢？我们不妨看下所继承的抽象类ClassLoader的源码。如果我们只是重写findClass方法，那么其loadClass(Sring str)依然保持原样，为：

```
public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);
}
```

我们再来看这个loadClass(name, false)：

```
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // 先检查目标类是否已被加载过了，避免重复加载
        Class c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
	        // 触发双亲委派模型，从父类加载器查起
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
            }

            // 父类加载器没找到
            if (c == null) {
                long t1 = System.nanoTime();
                c = findClass(name);    // 调用我们重写的findClass方法

                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

# 破坏双亲委派模型

除了前文提到的为了验证理论正确性这种可有可无的原因之外，碍于需求特殊，在官方历史中双亲委派模型主要出现过3次较大规模的被破坏的情况。

**第一次破坏：向前兼容**

双亲委派模型诞生于JDK1.2，而所有JVM之外的类加载器共同继承的抽象类ClassLoader则诞生于JDK1.1。在JDK1.2之前，开发人员都是直接重写loadClass方法以实现用户类加载器，因此时双亲委派模型尚未诞生，自然也不存在什么破坏不破坏之说。然而，在双亲委派模型诞生后，那些诞生于JDK1.1时代的，不遵循双亲委派模型的代码又必须兼容。本来，按照最简明干净的设计，因为双亲委派模型委派双亲的逻辑都在loadClass方法中，为防被破坏不应允许开发人员再轻易重写这个方法了，大家都重写官方推荐的findClass方法就好。然而为了兼容此前已经重写了这个方法的版本，又不得不允许重写：这就为破坏双亲委派模型留下了一个口子，其无法成为一个强制规范，而只能是希望开发人员自觉遵守的推荐。正如上文中我们所做的，很轻易的就将其破坏了。

**第二次破坏：上层类调用下层类**

首先我们先建立上层类与下层类的概念。因类加载器是有层级关系的，而类加载器+类信息又完全确定内存的一个类，因此这些内存中的类会因加载它们的类加载器而同样具有层级关系。被越是顶层的类加载器所加载的类的层级越高。Java的设计者将越是基础，越是核心的类用越是等层的类加载器加载，这很好理解：这些基础类是大哥，是要被用户类仰望，调用的存在。加载时按照双亲委派模型，优先用上层类加载器加载它们合情合理。

但是问题在于，有时大哥也需要调用小弟。或者更具体的说，上层类在加载时需要调用下层类，此时双亲委派模型就无法实现了。

一个比较典型的例子就是JNDI服务(类似的还有JDBC,JCE,JAXB,JBI等)。JNDI(Java Naming and Directory Interface,Java命名和目录接口)是SUN公司提供的一种标准的Java命名系统接口，JNDI提供统一的客户端API，并在运行时映射为不同的命名服务和目录系统。因这些实现都遵循相同的接口规范，Java应用程序便可以在这些命名服务和目录服务之间进行交互。JNDI服务是Java所提供的标准服务，其代码位于&lt;JAVA_HOME&gt;/lib目录下的rt.jar，由启动类加载器负责加载。但是由于JNDI服务的目的就是统合管理不同的命名和目录系统，因此其必然需要调用不同厂商提供的接口实现，即JNDI接口提供者(SPI,Service Provider Interface)。显然这些外部服务的代码所能使用的类加载器的最高层级也就到应用程序类加载器，其位置往往也都是位于ClassPath下。而启动类加载器则无法加载这些下层类。

为解决这个问题，Java设计者只好引入了一个不那么优雅的设计：线程上下文类加载器(Thread Context ClassLoader)。当创建线程A时，A会从父线程中继承到这个类加载器。若没有父类线程，则会默认填入应用程序类加载器。在线程创建后，开发人员也可以通过Thread类提供的setContextClassLoader方法设置自定义的用户类加载器。直白点说，启动类加载器不是读不到ClassPath下的第三方类嘛，那么就开个后门，再提供一个能读到这些类的类加载器，即该上下文类加载器。这样虽然能完成需求，但却相当于父类加载器请求子类加载器去完成类加载动作，打通了双亲委派模型的逆向调用顺序，是对双亲委派模型的一种破坏。

**第三次破坏：追求程序动态性**

所谓的动态性，就是指代码热替换(HotSwap)，模块热部署(Hot Deployment)等一系列花活。说白了，就是希望Java程序的代码能像电脑的键盘，鼠标，U盘这类支持热插拔的外设一样。插上就能用，想换直接换，而不用对电脑进行重启。这是有很重要的实际意义的。大型软件重启是一件代价很高的事：这意味着在此期间服务停止，或者为了维持服务可用不得不建立分布式备份。如果上下线能实现代码的热替换，将是一个重大的进步。

当前主流的Java动态性的支持来自于OSGi标准。在此我想多八卦几句，Sun在很早之前就认识到了模块化和动态性的重要性，相继提出了JSR-294，JSR-277规范(即Java模块系统，Java Module System)，然而遗憾的是市场却并不承认：在Java模块化规范这个问题上，Sun在与JCP组织的战争中败下阵来，市场公认的标准也是JCP组织的JSR-291(也就是大名鼎鼎的OSGi R4.2)。然而头铁是Sun一惯的作风，在Java这个语言上，SUN要的是绝对的统治力，所有关于Java的规范都得是自己制定的才行。无奈自己的标准实在是得不到认可，SUN便另起炉灶，你们不是不认可我的标准嘛？没事，我也不认可你们。这么想着SUN便独立搞起了Jigsaw项目继续做自己的标准。由此来看SUN这个公司的性格确实是有些问题，最终落得惨淡收场应该也是有很大的自身性格原因(和微软的那场世纪撕逼大战其实也类似)。

虽然名义上有两套标准了，但是我们要重点说的当然还是OSGi标准。OSGi(Open Service Gateway Initiative)是OSGi联盟(OSGi Alliance)制定的一个基于Java语言的动态模块化规范。这个规范最初由Sun(注意，最初是有Sun的。所以说Sun可真是...)，IBM，爱立信等公司联合发起，目的是使服务提供商通过住宅网关为各种家用智能设备提供各种服务(也就是OSGi这个名字的具体含义)，后来OSGi得到了不错的发展，其含义也得到扩展，现在已经成为了Java世界中"事实上"的动态模块化规范(之所以要打双引号，是因为虽然市场承认这个规范，然而大哥Sun不承认)。

OSGi已有诸如Equinox,Felix等成熟的实现，另外许多大型软件平台和中间件服务器都基于或已声明将会基于OSGi规范来实现，例如IBM Jazz平台，GlassFish服务器，jBoss OSGi等。同时后文欲讨论的热插拔技术基于的也是OSGi规范。不过对于普通的程序员而言，OSGi最著名的应用当属Eclipse IDE。

OSGi的每个模块(Bundle)与普通的Java类库的区别并不大，二者一般都以JAR格式封装，内部存储的也都是Java Package和Class。然而OSGi的模块有着更高的灵活性：它可以通过Import-Package声明自身所依赖的Java Package，也可以通过Export-Package声明它允许导出发布的Java Package。这样可以使得类的访问权限更为精确，未被Export的Package及Class均会被隐藏起来。

对于标准的Java应用而言，类库间的依赖是有明显的层级关系的：程序员所编写的代码都依赖于Java的核心类库。而OSGi则打破了这种层级关系，至少从外观上来看，因为OSGi的模块本身基本是平级的，其模块间的依赖自然也是平级关系的依赖。

OSGi之所以能有上述区别于Java标准应用的特点，主要源于它那灵活的类加载器架构。

OSGi完全颠覆了Sun推荐的双亲委派模型。在OSGi中，每个模块都有一个属于自身的类加载器，而各类加载器之间只有规则，没有固定的委派关系。正因为模块与类加载器是这种一对一的关系，那么上面那句话我们也可以反过来说：每个类加载器都负责管理加载一个模块，所有对这个模块中的Package或Class的加载都由该类加载器完成。这样根据每次具体代码的不同，都会形成不同的临时的类加载器层次结构。或者更明确的说，在OSGi环境下，类加载器的层级结构不再是双亲委派模型中的树状结构，而是发展为了更复杂的网状结构。简单来说，除了最核心的那些类有点层级关系之外，绝大多数的类都是在平级的类加载器中完成的。

OSGi标准实现模块化热部署的基石也正是它的类加载机制：每个程序模块都有一个属于自己的类加载器，当需要做热替换更换一个Bundle时，会把该Bundle连同其类加载器一起换掉。

# Tomcat中的类加载器

主流的Java Web服务器，如Tomcat，Jetty，WebLogic，WebSphere等都会实现自定义类加载器，而且通常都不止一个。这主要是由Web服务器的功能需求决定的：

- 在JVM眼中，所谓的Web服务器也不过是一个普通的Java程序(这里指那些由Java写成的Web服务器)，并没有什么特殊的。然而在程序员眼中Web服务器却是部署应用的平台：它不该是程序，它应该是程序的容器。这几乎是后面我们要论述的一切矛盾的根源。
- 既然我们认为Web服务器是部署应用的平台，那么一个最基本的需求就是部署在同一个Web服务器上的两个应用程序应该实现代码隔离，或者起码来说，要让程序员看起来是这么回事。例如两个不同的应用可能会同时依赖于同一个第三方类库(也就是我们一般意义上的jar包)的不同版本，而因为是同一个类库，因此该类库绝大多数的代码，包括包路径，类路径等等都是相同。此时服务器不仅仅要同时加载这两个版本，并且还能明确知道哪个版本是给哪个应用用的。
- 我们再来说说上一条需求的反面。实际开发中，若服务器上的两个应用用到了同一个类库，那么使用不同版本的可能性其实不大。绝大多数情况下都会使用相同的版本。此时就涉及到共享的问题了。当然，我们也可以继续沿用上个需求的做法，即便版本相同，也为各个应用独立加载类库。从磁盘存储的角度上讲，这样做的影响一般不大(毕竟硬盘通常都是很大的，多放几个jar包基本没什么问题)。但是内存的消耗就有些大了，独立加载意味着多加载了好多重复的类到内存，而内存就没硬盘那么廉价了。因此代码的共享问题也是Web服务器必须考虑的。
- 很多运行Java应用的Web服务器本身也是用Java编写的，服务器程序本身也存在类库依赖问题。那么对于这类Web服务器而言，还需要考虑安全问题。因为显然不能让应用的代码影响到服务器程序本身，要挂也是只挂你这个应用本身，而不是将整个平台拖垮。因此服务器所使用的类库应该与应用的类库隔离。
- 既然要运行Java应用，那么就要支持JSP。不过JSP文件最终还是要编译为Class文件才能在JVM中执行，而与其它正经的由.java生成的Class文件相比，JSP在运行时修改的概率要大得多。而在网页应用的圈子里(比如ASP,PHP,JSP等)，这些页面基本是被人当做Html去看待的(这也是它们诞生的目的和原因)，那么修改这些页面而无需重启应用就是一件理所当然的事，如果某个页面做不到这一点，那么它基本也就别想在这个圈子里混了。因此既然要支持JSP，那么通常就要支持JSP生成类的热替换(HotSwap)功能。这里既然说到了通常，那么当然有不通常的情况：例如运行在生产模式(Production Mode)下的WebLogic服务器默认就不会处理JSP文件的变化。

正是因为这些原因，单靠JVM所提供的那个应用程序类加载器显然就不够了。因此各种Java Web服务器都会不约而同的设计多个自定义类加载器。反映到表象上，就是Java Web服务器通常都会提供好几个ClassPath路径供用户应用存放自身用到的第三方类库，这些路径通常都会以lib或classes命名。被放置到不同路径下的类库，自然会有不同的访问范围和服务范围。通常，每个路径都会有一个相应的自定义类加载器负责加载放置于其中的类库。

至此，程序员对Java Web服务器在类加载器方面的需求算是大概介绍完了。而为了实现这个相同的需求，各服务器会采取不同的做法，这里我们要聊的是Tomcat。

作为Apache基金会中的一款开源的Java Web服务器，Tomcat基本上遵循了双亲委派模型。本文中用于举例的Tomcat的版本为apache-tomcat-7.0.82。其目录结构如下：

![1.jpg](/images/blog_pic/JVM/类加载器/1.jpg)

其类加载架构为：

![2.jpg](/images/blog_pic/JVM/类加载器/2.jpg)

其中蓝底部分为JVM预设的类加载器，而白底部分则是Tomcat的自定义类加载器。

首先先来看Common这个自定义类加载器，其所加载的类路径存在conf/catalina.properties中，其默认值为：

```
common.loader=${catalina.base}/lib,${catalina.base}/lib/*.jar,${catalina.home}/lib,${catalina.home}/lib/*.jar
```

关于Tomcat中catalina.base及catalina.home的区别，详见[Tomcat-CATALINA_HOME与CATALINA_BASE](/2017/12/12/Tomcat-CATALINA_HOME与CATALINA_BASE/)

然后我们再来看看这个lib目录下到底有什么：

![3.jpg](/images/blog_pic/JVM/类加载器/3.jpg)

很显然，这是Tomcat容器级别所依赖的类库。这里我想多说一句的是，很多程序员在用Eclipse之类的IDE编写Servlet程序时会提示缺少servlet-api.jar的依赖，然而即便不管它，将程序打包放到Tomcat后依然可以正确运行。其原因就在于Tomcat已将整个类库加到了其Common类加载器的ClassPath中。

显然，Common类加载器的ClassPath中应该放置Tomcat平台级别的类库，因为它是所有Webapp类加载器的父类加载器。

接着我们继续来看再下一层的Webapp类加载器。显然，它们是应用私有的。每个应用都有一个独立的，与之对应的自定义类加载器。其加载的类库范围为：

- webapps/用户目录/WEB-INF/classes目录下的用户代码
- webapps/用户目录/WEB-INF/lib目录下应用程序所依赖的类库

因为要保证前文提到的需求，这些类加载器就需要破坏双亲委派模型。因此我们才说Tomcat"基本上"遵循了双亲委派模型。对于具体一个类，其类加载器的优先顺序为：

```
启动类加载器
扩展类加载器
应用程序类加载器
Webapp
Common
```

显然，对于JVM预设的那一层次的类加载器而言，Tomcat依然是严格遵循双亲委派模型的。然而到了Tomcat自定义的类加载器这一层，双亲委派模型遭到了破坏：会优先使用应用本身的Webapp类加载器，随后才是Tomcat平台的Common类加载器。