---
title: Java 设计模式-13.Visitor模式
date: 2018-08-08 18:24:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Visitor模式被归入了第6部分[访问数据结构]()。在GoF原书中，Visitor模式则被归入了[行为型设计模式]()。简单来说，Visitor模式可以被描述为：将数据结构的存储与处理分离开。

<!-- more -->

# 综述

所谓数据结构，就是指以某种特定结构存储的数据，不过显然，存储并不是目的，使用与处理才是。通常，我们会将处理数据结构的代码直接写在描述数据结构的代码中，这样的好处是直观简单，坏处则是违背了开闭原则，因为只要设计得得当，通常我们是不会修改数据结构的存储结构本身的，我们会做的往往只是添加操作它的方法。因此随着时间的增加，我们对该数据结构的操作会越来越多，越来越复杂，还将存储与操作集中在一个位置显然不利于代码的维护。

Visitor模式应运而生。Visitor是"访问者"的意思。简单来说，在这种模式下，数据结构是静态的仓库，只用于数据的存储。相应的，数据处理的功能被转移至访问者，从而实现了数据结构存储与处理的分离。当我们需要增加新的处理方法时，无需修改数据结构的代码本身，只需要增加新的访问者即可。

# 示例程序

下面我们给出一个小例子，首先是类图：

![0.jpg](/images/blog_pic/Java 设计模式/13Visitor模式/0.jpg)

本程序中的所有代码将被统一置于design13包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/13Visitor模式/1.jpg)

其中Main.java是测试代码，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Warrior类**

```
package design13;

public class Warrior {

    private String name;

    private String race;

    private int attack;

    private int defense;

    public Warrior(String name, String race, int attack, int defense) {
        this.name = name;
        this.race = race;
        this.attack = attack;
        this.defense = defense;
    }

    public void show(Visitor visitor) {
        visitor.show(this);
    }

    /**
     * 随机增强或减弱
     * @param max int, 增强或减弱的上限
     */
    public void change(Visitor visitor, int max) {
        visitor.change(this, max);
    }

    public String getName() {
        return name;
    }

    public String getRace() {
        return race;
    }

    public int getAttack() {
        return attack;
    }

    public void setAttack(int attack) {
        this.attack = attack;
    }

    public int getDefense() {
        return defense;
    }

    public void setDefense(int defense) {
        this.defense = defense;
    }
}
```

**Visitor类**

```
package design13;

import java.util.Random;

public class Visitor {

    private Random random = new Random();

    public void show(Warrior warrior) {
        System.out.println("[" + warrior.getName() + "][" + warrior.getRace() + "][攻击力：" + warrior.getAttack() + "][防御力：" + warrior.getDefense() + "]");
    }

    public void change(Warrior warrior, int max) {
        int flag = this.random.nextInt(2);
        if (flag == 0) {
            System.out.println("恭喜~属性增强！所有属性提高" + max);
            warrior.setAttack(warrior.getAttack() + max);
            warrior.setDefense(warrior.getDefense() + max);
        } else {
            System.out.println("真不幸~属性降低！所有属性减少" + max);
            warrior.setAttack(warrior.getAttack() - max);
            warrior.setDefense(warrior.getDefense() - max);
        }
    }
}
```

**Main类**

```
package design13;

public class Main {

    public static void main(String[] args) {
        Warrior warrior = new Warrior("博丽灵梦", "人类", 97, 85);
        Visitor visitor = new Visitor();
        warrior.show(visitor);
        warrior.change(visitor, 10);
        warrior.show(visitor);
    }
}
```

执行后，可能的输出为：

```
[博丽灵梦][人类][攻击力：97][防御力：85]
恭喜~属性增强！所有属性提高10
[博丽灵梦][人类][攻击力：107][防御力：95]
```

# 登场角色

上面的示例程序介绍了Visitor模式的Java实现，下面咱们试着跳出语言层面，抽象出Visitor模式中登场的角色。

**Structure(数据结构)**

在示例程序中，由Warrior类扮演这个角色。

**Visitor(访问者)**

在示例程序中，由Visitor类扮演这个角色。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/13Visitor模式/2.jpg)

# 一些说明

在GoF的23种设计模式中，Visitor模式确实是比较特殊的。这从它的类图就能看出来，Visitor模式中只有两个角色(这倒没什么)，而这两个角色之间是没有直接的关联的：不仅没有继承这种强联系，甚至连委托这种弱关系也没有。

那么Visitor模式的调用关系是什么样子的呢？

以示例程序中Warrior类的change()方法为例，该方法的作用是修改Warrior的属性。从调用者Main的角度来看，它是这样做的：

```
warrior.change(visitor, 10);
```

也就是说，在它看来，问题依然还是warrior这个数据结构本身处理的，它并不知道有"数据结构的存储和操作分离"这回事(它也不care这事)。不过和最简单的调用相比，它还是多传入了一个visitor。而在change()方法内部，实际进行数据处理的是visitor，而由于被处理的对象是warrior，warrior还需把visitor处理问题时需要的数据传递给visitor(示例程序采用的是比较粗暴的方式，直接将this，也就是warrior本身传递给visitor了。实际情况往往只需要传递warrior的子集数据即可)。

不得不说，这和委托很像：调用者想要处理Structure，但是Structure本身没有处理能力，调用者也没有这个处理能力。因此调用者就找来了具有处理能力的Visitor，让它来协助Structure。从调用者来看，处理数据的依然是Structure，只不过在它的内部，实际的处理工作被委托给了Visitor。而Visitor如果想处理数据，往往又需要Structure提供必要的信息。

当然，这又不是委托。因为Structure同Visitor之间没有任何关系：Structure中并没有一个字段是Visitor类型的。在编程领域，这种调过来又调过去的做法被称为双重分发。

那么为什么使用双重分发而非委托呢？这是因为双重分发更为灵活。此时Visitor并没有与Structure绑定，这意味着我们在需要的时候可以很灵活的替换Visitor，通常来说，我们会声明一个抽象的Visitor，限定一下Visitor必须要遵循的规范，当想要进行新的操作时，只需要编写新的Visitor即可，而Structure则无需做任何修改，显然，这是遵循[开闭原则]()的。

话说回来，Visitor模式相当于是将通常的Structure一分为二：即只有存储功能的Structure，和负责操作数据的Visitor。上文讨论了Visitor的替换，下面再来说说Structure的替换：相对而言，Structure的替换和修改就不那么容易了，因为如果要替换Structure，那么所有已有的Visitor都需要修改。这又是一个老问题了，我们在[8.Abstract Factory模式]()也讨论过类似的困境。

因此，我们通常只会修改和新增Visitor，而不会动Structure。这是因为Structure是基础，所有的Visitor都是附属于Structure的。所谓"皮之不存毛将焉附"。其实从广义的角度来讲，虽然具体架构迥异，但是[1.Iterator模式]()也算是一种Visitor模式的一种了。之所以这么说，是因为Visitor模式的核心点其实只有一个，那就是"将数据结构的存储与操作分离"，而Iterator模式的目标实际就是在将迭代操作自容器结构中分离出去。

提到分离，我们不禁又会想到另一种以分离为目标的设计模式：[9.Bridge模式]()，二者在各个方面均差异巨大，不过其本源的思想却是一致的，那就是将复杂的结构和操作尽量的分解，从而让逻辑变得清晰。

# 相关设计模式

**[1.Iterator模式]()**

从广义的角度来讲，Iterator模式是Visitor模式的特例。

Visitor模式的目的是将处理自数据结构中分离出来，而Iterator模式的目的则是将迭代操作自容器结构中分离出来。

---

**[8.Abstract Factory模式]()**

二者均既遵循又违背了开闭原则。

Abstract Factory模式易于扩展具体的工厂，此时遵循开闭原则。难以增加新的零件，此时违背开闭原则。

Visitor模式易于增加新的Visitor，此时遵循开闭原则。难以修改Structure，此时违背开闭原则。

---

**[9.Bridge模式]()**

二者的核心思想均为"分离"。