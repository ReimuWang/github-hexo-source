---
title: Java 设计模式-4.Factory Method模式
date: 2018-06-14 18:44:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Factory Method模式被归入了第2部分[交给子类]()。在GoF原书中，Factory Method模式则被归入了[创建型设计模式]()。简单来说，Factory Method模式可以被描述为：在抽象的父类工厂中定义生成抽象的父类产品的逻辑，然后由具体的子类工厂生产出具体的子类产品。也就是所谓的"将实例的生成交给子类"。

<!-- more -->

# 综述

简单来说，我们可以认为Factory Method模式是[3.Template Method模式]()的特例。Template Method模式是指父类定义流程，却不负责具体的实现。而Factory Method模式则是进一步将流程限定为"实例的生成"。

因此，后文在介绍Factory Method模式时，基本都会对照着Template Method模式来说。

# 示例程序

本程序会分种族的使用工厂创建东方Project的角色，下面先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/4Factory Method模式/0.jpg)

本程序中的所有代码将被统一置于design4包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/4Factory Method模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**RoleFactory类**

```
package design4;

public abstract class RoleFactory {

    public Role create(String name) {
        return this.createRole(name);
    }

    protected abstract Role createRole(String name);
}
```

其中，create()方法就是Template Method模式中的模版方法，而createRole()方法则是Template Method模式中需子类实现的抽象方法。

**Role类**

```
package design4;

public abstract class Role {

    private String name;

    protected Role(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "我叫" + this.name + "，我的种族是" + this.race();
    }

    protected abstract String race();
}
```

**HumanFactory类**

```
package design4;

public class HumanFactory extends RoleFactory {

    @Override
    protected Role createRole(String name) {
        return new Human(name);
    }
}
```

**Human类**

```
package design4;

public class Human extends Role {

    protected Human(String name) {
        super(name);
    }

    @Override
    protected String race() {
        return "人类";
    }
}
```

**VampireFactory类**

```
package design4;

public class VampireFactory extends RoleFactory {

    @Override
    protected Role createRole(String name) {
        return new Vampire(name);
    }
}
```

**Vampire类**

```
package design4;

public class Vampire extends Role {

    protected Vampire(String name) {
        super(name);
    }

    @Override
    protected String race() {
        return "吸血鬼";
    }
}
```

**Main类**

```
package design4;

import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        List<Role> roleList = new ArrayList<Role>(5);
        RoleFactory roleFactory = new HumanFactory();
        roleList.add(roleFactory.create("博丽灵梦"));
        roleList.add(roleFactory.create("雾雨魔理沙"));
        roleList.add(roleFactory.create("十六夜咲夜"));
        roleFactory = new VampireFactory();
        roleList.add(roleFactory.create("蕾米莉亚·斯卡雷特"));
        roleList.add(roleFactory.create("芙兰朵露·斯卡蕾特"));
        for (Role role : roleList) System.out.println(role);
    }
}
```

执行Main.java后输出：

```
我叫博丽灵梦，我的种族是人类
我叫雾雨魔理沙，我的种族是人类
我叫十六夜咲夜，我的种族是人类
我叫蕾米莉亚·斯卡雷特，我的种族是吸血鬼
我叫芙兰朵露·斯卡蕾特，我的种族是吸血鬼
```

# 登场角色

上面的示例程序介绍了Factory Method模式的Java实现，下面咱们试着跳出语言层面，抽象出Factory Method模式中登场的角色。

**Creator(创建者)**

Creator对应于Template Method模式中的抽象父类，其中至少应包含两个方法，首先是create()方法，它对应于Template Method模式中的模版方法，用来控制"生成实例"这个业务流程。然后是factoryMethod()方法，它对应于Template Method模式中的抽象方法，需要子类角色ConcreteCreator去具体的实现。

在示例程序中，抽象类RoleFactory扮演了这个角色。

**Product(产品)**

Product是Creator创建出的实例所属的类。当然，Creator是抽象的工厂，是无法创建实例的。因此Product是抽象的产品。

Creator与Product均是抽象上的概念，它们之间是互相对应的，不承载具体的业务逻辑。Creator就是抽象的生成产品的工厂，至于具体是生成什么产品的，它不知道。而Product是抽象意义上的产品，它不是某种具体的产品，它代表的是"产品"这个概念。

在示例程序中，抽象类Role扮演了这个角色。当然啦，在示例程序中，因为Role类中包含抽象方法race()，因此必须要被声明为抽象的。但实际上，Product中并非必须要包含抽象方法，所以仅从Java语言规范的角度来讲，此时Product也可以是非抽象的。

至于是否要将Product声明为抽象的，则要依具体的情境而定。如果Product非常抽象，真的就只是代表一个概念，绝不会被实例化，那么建议大家将Product声明为抽象的(即便其中没有抽象方法)，从语法的角度上讲，这可以保证Product不会被new出实例。更重要的是，这表明了一种态度：Product代表的是抽象的"产品"这一概念，而不是某个具体的产品。反之，如果Product虽然也代表"产品"这个概念，但是它却没有那么抽象，我们甚至可以实例化一个代表"产品"的实例出来，那么自然就不能声明为抽象的了。

**ConcreteCreator(具体的创建者)**

ConcreteCreator对应于Template Method模式中的子类，它会实现其抽象父类Creator中的抽象方法。在示例程序中，HumanFactory及VampireFactory均饰演这个角色。作为工厂它们分别负责生成具体的产品Human及Vampire。

之所以创建了两个工厂，是为了说明抽象与具体之间的关系：抽象角色(Creator,Product)是唯一的，Creator代表"工厂"这个概念，而"Product"代表"产品"这个概念，概念自然是唯一的。但是属于这个概念的个体则是无穷多个的，在本程序中，就有生产人类的工厂及生产吸血鬼的工厂。如果我们需要，还可以很轻松的添加其他的工厂：诸如生产魔法使的工厂及生产神的工厂等。

**ConcreteProduct(具体的产品)**

ConcreteProduct与ConcreteCreator是一一对应的。什么样的工厂就会生产什么样的产品。在示例程序中，Human及Vampire均扮演了这个角色。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/4Factory Method模式/2.jpg)

# Factory Method模式的变种

除了实例代码中的RoleFactory类之外，Creator这个角色其实还可以这样写：

```
public class TestFactory {

    public TestProduct create() {
        return this.createTest();
    }

    protected TestProduct createTest() {
        return new TestProduct();
    }
}
```

较之前文中的示例程序，这段代码中的Creator角色其实就不完全遵循Template Method模式了。此时的Factory Method模式(假设我们还认为这样的变种算是Factory Method模式的话)就不再是Template Method模式的特例了，因为createTest()方法不再是抽象的了。相应的，因为需要切实的实例化，作为抽象产品的TestProduct类自然不能再被声明为abstract的了，这也就是前文我们在介绍Product角色时提到的"不那么抽象的情况"。

这样做的好处和坏处都是显而易见的。从好的角度来讲，此时抽象工厂实际上是有生产能力的了，所以从严格的意义上讲，TestFactory不应该被叫做抽象工厂，而应该被叫做默认工厂。相应的，Product也应该被叫做默认产品。在子类没有什么特殊需要的情况下，它们直接用父类提供的统一规格的产品就行。如果它们有特殊的需求，那么重写createTest()方法就好。

而从坏的角度上讲，这相当于是"坏了规矩"。此时的Creator及Product不再代表一个概念了，必须是非抽象的了，因为无论它们被声明的多通用化，它们都是有实体的了，是要产生实例的。而子类由重写父类抽象方法到重写父类非抽象方法，其强制力也进一步降低：前者是非做不可，而后者则是可做可不做。

说"坏了规矩"，貌似是个挺严重的事。不过在实际编程中，规矩有时也没那么重要，根据实际的情况，做出最合理的应对才是最重要的。事实上，并不仅仅是Factory Method模式，所有的设计模式说穿了都是经验的总结，并不是规范性的东西，因此一切设计模式，其实都应该根据实际情况做出灵活的应对。

# 相关设计模式

**[3.Template Method模式]()**
**[7.Builder模式]()**
**[8.Abstract Factory模式]()**

关于这4个设计模式间的联系与区别，详见[8.Abstract Factory模式]()。

---

**[5.Singleton模式]()**

Creator与ConcreteCreator这个角色通常都是唯一的。Creator的唯一性比较好理解，表示"工厂"的概念只有一个，Creator自然只需要一个就够了。需要进一步解释一下的是ConcreteCreator这个角色。具体的工厂可以有多个类型，但是每个类型的工厂通常只需要一个。例如我们可以同时创建生产桔子与生产苹果的工厂。从总数上说，扮演ConcreteCreator角色的实例有两个，但具体到每种特定的类型，则只会有一个。

正因为这种唯一性，Creator与ConcreteCreator通常都会应用单例模式来确保单一性。实际上，并不仅仅是这两个角色，也不仅仅是局限在Factory Method模式，任何需要从逻辑上确保单一性的角色都可以应用单例模式。

当然，我们的示例程序仅仅只是举例，因此不想弄得太复杂，自然就没使用单例模式啦。

---

**[1.Iterator模式]()**

我们可以应用Factory Method模式来实现Iterator模式。角色对应如下：

- Creator：生产"迭代器"这个概念的抽象工厂。
- Product：代表"迭代器"这个概念的抽象类。
- ConcreteCreator：生成具体迭代器的工厂。
- ConcreteProduct：具体的迭代器，例如链表的迭代器，数组的迭代器等。