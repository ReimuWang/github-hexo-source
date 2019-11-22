---
title: Java 设计模式-12.Decorator模式
date: 2018-08-06 18:28:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Decorator模式被归入了第5部分[一致性]()。在GoF原书中，Decorator模式则被归入了[结构型设计模式]()。简单来说，Decorator模式可以被描述为：使装饰边框与被装饰物具有一致性。

<!-- more -->

# 综述

假设有一块蛋糕，它可以增加食用者的饱腹度及幸福感。当然，我们可以给这块蛋糕加一些装饰：比如，如果我们把草莓放在蛋糕上，那么它就变为了草莓蛋糕；如果我们在蛋糕外面淋上巧克力，那么它就变为了巧克力蛋糕，等等。当然，这些装饰是可以叠加的：比如，如果我们既在蛋糕外面淋上巧克力，又在蛋糕上面放了草莓，那么它就变为了草莓巧克力蛋糕(把自己写饿了...)。

不过，不管我们如何添加装饰，它本质上还是一块蛋糕，它的核心功能并没有变，依然还是增加食用者的饱腹度及幸福感，只不过具体增加的数值会依装饰物的不同而有所变化。

类比到编程领域，由这种思路演化而来的设计模式就是Decorator模式(decorator的含义是装饰物)。在Decorator模式中，首先会有一个被装饰物，它提供了核心的功能(或者说API)，然后我们对其添加装饰物，正如蛋糕放上草莓后依然还是蛋糕那样，被装饰物经过装饰后种类没有变，依然还是原被装饰物。这样，当我们需要添加新的装饰时，就可以在此基础上依同样的方法进行操作。从而在不改变最初的被装饰物代码的前提下，丰富它的功能。

# 示例程序

下面我们将综述中草莓蛋糕的小例子扩展为程序，从而说明Decorator模式的用法。

首先我们定义一个抽象类Dessert，用以表示甜点的概念，作为Decorator模式中的被装饰物。

然后我们定义具体的两种甜点：

- 蛋糕：Cake
- 冰淇淋：IceCream

然后我们再定义抽象类Decorator，顾名思义，它表示用来装饰甜点的添加物的概念。下面重点来了：Decorator中有一个Dessert类型的字段，这样它就可以通过委托的方式调用自身装饰的被装饰物的功能。同时Decorator又继承了Dessert，这样它在装饰完自己要装饰的被装饰物后，自身同样可以作为被装饰物再被其他的装饰物装饰(艾玛好绕)。

下面定义两个具体的装饰物：

- 巧克力：Chocolate
- 草莓：Strawberry

下面来详细介绍代码。首先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/12Decorator模式/0.jpg)

本程序中的所有代码将被统一置于design12包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/12Decorator模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Dessert类**

```
package design12;

public abstract class Dessert {

    /**
     * 饱腹感
     */
    protected abstract int getSatiety();

    /**
     * 幸福度
     */
    protected abstract int getHappiness();

    protected abstract String name();

    @Override
    public String toString() {
        return "成分：" + this.name() + "\n饱腹感：" + this.getSatiety() + "，幸福度：" + this.getHappiness();
    }
}
```

这里的toString()方法应用了[3.Template Method模式]()。

**Cake类**

```
package design12;

public class Cake extends Dessert {

    @Override
    protected int getSatiety() {
        return 10;
    }

    @Override
    protected int getHappiness() {
        return 5;
    }

    @Override
    protected String name() {
        return "[蛋糕]";
    }
}
```

**IceCream类**

```
package design12;

public class IceCream extends Dessert {

    @Override
    protected int getSatiety() {
        return 5;
    }

    @Override
    protected int getHappiness() {
        return 10;
    }

    @Override
    protected String name() {
        return "[冰淇淋]";
    }
}
```

**Decorator类**

```
package design12;

public abstract class Decorator extends Dessert {

    protected Dessert dessert;

    protected Decorator(Dessert dessert) {
        this.dessert = dessert;
    }
}
```

**Chocolate类**

```
package design12;

public class Chocolate extends Decorator {

    public Chocolate(Dessert dessert) {
        super(dessert);
    }

    @Override
    protected int getSatiety() {
        return this.dessert.getSatiety() + 3;
    }

    @Override
    protected int getHappiness() {
        return this.dessert.getHappiness() + 3;
    }

    @Override
    protected String name() {
        return this.dessert.name() + "[巧克力]";
    }
}
```

**Strawberry类**

```
package design12;

public class Strawberry extends Decorator {

    public Strawberry(Dessert dessert) {
        super(dessert);
    }

    @Override
    protected int getSatiety() {
        return this.dessert.getSatiety() + 2;
    }

    @Override
    protected int getHappiness() {
        return this.dessert.getHappiness() + 2;
    }

    @Override
    protected String name() {
        return this.dessert.name() + "[草莓]";
    }
}
```

**Main.java**

首先，我们测试只有被装饰物的情况：

```
package design12;

public class Main {

    public static void main(String[] args) {
        Dessert cake = new Cake();
        System.out.println(cake);
    }
}
```

输出：

```
成分：[蛋糕]
饱腹感：10，幸福度：5
```

然后我们可以给这个蛋糕加一些装饰：

```
package design12;

public class Main {

    public static void main(String[] args) {
        Dessert cake = new Strawberry(new Chocolate(new Cake()));
        System.out.println(cake);
    }
}
```

输出：

```
成分：[蛋糕][巧克力][草莓]
饱腹感：15，幸福度：10
```

首先，虽然具体new的东西变了，但是引用类型并没有变，依然还是Dessert，或者更具体的说，因为最底层的被修饰物是蛋糕，因此最终得到的依然还是蛋糕。上例中我们先给蛋糕装饰了巧克力，后装饰了草莓。当然我们也可以先装饰草莓，后装饰巧克力：

```
package design12;

public class Main {

    public static void main(String[] args) {
        Dessert cake = new Chocolate(new Strawberry(new Cake()));
        System.out.println(cake);
    }
}
```

输出：

```
成分：[蛋糕][草莓][巧克力]
饱腹感：15，幸福度：10
```

在这个小例子中，这样的变换似乎意义不大。不过在很多场景中，交换装饰顺序是能在很大程度上影响最终效果的。

当然啦，我们也可以将被装饰物由蛋糕换为同为甜点的冰淇淋。所有的操作都是一样的，在此仅给出一个小例子：

```
package design12;

public class Main {

    public static void main(String[] args) {
        Dessert cake = new Chocolate(new Strawberry(new IceCream()));
        System.out.println(cake);
    }
}
```

输出：

```
成分：[冰淇淋][草莓][巧克力]
饱腹感：10，幸福度：15
```

# 登场角色

上面的示例程序介绍了Decorator模式的Java实现，下面咱们试着跳出语言层面，抽象出Decorator模式中登场的角色。

**Component(抽象的被装饰物)**

在示例程序中，由Dessert类扮演这个角色。

**ConcreteComponent(具体的被装饰物)**

在示例程序中，由Cake类及IceCream类联袂扮演这个角色。

**Decorator(抽象的装饰物)**

Decorator模式的核心角色。它内部保存了Component，这样就可以通过委托调用被装饰物的功能。同时它又继承了被装饰物，这样它又可以作为被装饰物再被其他装饰物装饰，即被装饰物的API并不会因装饰物而隐藏，在编程领域，这被称为API的透明性。在示例程序中，由Decorator类扮演这个角色。

**ConcreteDecorator(具体的装饰物)**

在示例程序中，由Chocolate类及Strawberry类联袂扮演这个角色。

在Decorator模式中，虽然核心功能来源于Component，但我们却可以通过Decorator很方便的为Component增加功能，这是Decorator模式的优点，不过这样做会增加许多功能类似的很小的类(也就是ConcreteDecorator)，算是白璧微瑕。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/12Decorator模式/2.jpg)

# 相关设计模式

**[11.Composite模式]()**

二者均保证了不同类间的一致性，从而可以以递归的方式去操作它们。

# java.io包对Decorator模式的应用

在使用java.io包时，我们可以按如下方式生成一个读取文件的实例：

```
Reader reader = new FileReader("a.txt");
```

下面我们来分析下这条语句，在分析过程出现的类或接口，除了特殊说明以外，均位于java.io包中。

首先是FileReader，它的类定义如下：

```
public class FileReader extends InputStreamReader
```

而这个InputStreamReader的类定义为：

```
public class InputStreamReader extends Reader
```

果然，它继承了Reader类，因此我们才能将FileReader的实例赋给Reader。而Reader的类定义为：

```
public abstract class Reader implements Readable, Closeable
```

这个Reader就是本次讨论的顶点类了，相当于Decorator模式中的Component。而FileReader及InputStreamReader则均属于Decorator模式中的ConcreteComponent。

按照这种方法，我们可以得到一个比较基本的读取文件的实例。在此基础上，如果我们想增加一些功能，例如在读取文件时将其放入缓冲区，那么可以这样做：

```
Reader reader = new BufferedReader(new FileReader("a.txt"));
```

显然，这个BufferedReader应该就是Decorator模式中的ConcreteDecorator了。我们来看一下它的类定义：

```
public class BufferedReader extends Reader
```

这里需要注意一下，BufferedReader直接继承了Reader，也就是说此时是没有Decorator这个角色的，关于这一点，此前我们在讨论其他设计模式时已经说过很多次了：设计模式并非是一个死的标准，在使用时应根据具体情况做灵活的变通。具体这本场景，业务逻辑并没有那么复杂，即便去掉Decorator这一层代码结构依然很清晰，不存在Decorator也是很合理的做法。

而在BufferedReader类中，我们果不其然的找到了这样的字段：

```
private Reader in;
```

很显然，这就是被委托的被装饰物了。为了印证这一点，我们不妨看一下所调用的构造函数的源码：

```
public BufferedReader(Reader in) {
    this(in, defaultCharBufferSize);
}
```

它内部调用的那个构造函数的源码为：

```
public BufferedReader(Reader in, int sz) {
    super(in);
    if (sz <= 0)
        throw new IllegalArgumentException("Buffer size <= 0");
    this.in = in;
    cb = new char[sz];
    nextChar = nChars = 0;
}
```

与我们的预期相符。

当然，ConcreteDecorator一般不会只有一个，如果我们还需要增加管理行号的功能，那么可以这样做：

```
Reader reader = new LineNumberReader(new BufferedReader(new FileReader("a.txt")));
```

而这个LineNumberReader的类定义为：

```
public class LineNumberReader extends BufferedReader
```

哦，原来它是BufferedReader的子类，这里就不再赘述了。