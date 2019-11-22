---
title: Java 设计模式-19.State模式
date: 2018-08-31 20:25:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，State模式被归入了第8部分[管理状态]()。在GoF原书中，State模式则被归入了[行为型设计模式]()。

<!-- more -->

# 综述

在面向对象编程领域，有这样一句话：一切都是对象，对象就是一切。这话虽说有些绝对了，但也从一个侧面说明对象在面向对象编程领域的普遍性。在我们通常的认知中，对象指的是某个具体的事物，例如一张桌子，一把椅子。但是对象其实还可以表示一些抽象的事物，例如本文要提到的"状态"。

漫画《七龙珠》的主人公是卡卡罗特(孙悟空)。将卡卡罗特看作一个对象是比较好理解的。但我们同样也可以将卡卡罗特所处于的状态视为对象。例如我们可以将普通状态视为一个类，超级赛亚人状态视为另一个类。这样再描述卡卡罗特所处于的状态时，就可以用具体状态的实例来表示。

普通状态(左)与超级赛亚人状态(右)下的卡卡罗特：

![0.jpg](/images/blog_pic/Java 设计模式/19State模式/0.jpg)

在编程领域，由此演化而来的设计模式就是State模式。以类来表示状态后，我们就能通过切换表示状态的类来方便的改变对象所处于的状态。同时，当需要增加新的状态时(例如为卡卡罗特再增加一个巨猿状态)，条理也会更为清晰。

# 示例需求描述

狂野女猎手-奈德丽(Nidalee)是一位出自游戏英雄联盟(LOL)的女英雄，在描述具体的需求之前，先展示几张她的皮肤原画吧~

**默认**

![1.jpg](/images/blog_pic/Java 设计模式/19State模式/1.jpg)

---

**雪装**

![2.jpg](/images/blog_pic/Java 设计模式/19State模式/2.jpg)

---

**丛林猎豹**

![3.jpg](/images/blog_pic/Java 设计模式/19State模式/3.jpg)

---

**法国女仆**

![4.jpg](/images/blog_pic/Java 设计模式/19State模式/4.jpg)

---

**艳后**

![5.jpg](/images/blog_pic/Java 设计模式/19State模式/5.jpg)

---

**魅惑女巫(2011年万圣节)**

![6.jpg](/images/blog_pic/Java 设计模式/19State模式/6.jpg)

---

**枭姬 孙尚香**

![7.jpg](/images/blog_pic/Java 设计模式/19State模式/7.jpg)

---

**勇者**

![8.jpg](/images/blog_pic/Java 设计模式/19State模式/8.jpg)

好漂亮啊有没有，尤其是枭姬，英姿勃发。不过我们之所以介绍得这么详细并不完全是因为她生得俊俏(你够了)，而是因为她是具有双形态的英雄。下面就以枭姬这个皮肤的3D模型来展示一下：

**人形态**

![9.jpg](/images/blog_pic/Java 设计模式/19State模式/9.jpg)

---

**豹形态**

![10.jpg](/images/blog_pic/Java 设计模式/19State模式/10.jpg)

奈德丽的R技能为形态切换。在各自的形态下，她有着独立的两套QWE技能(传说中拥有7个主动技能的女人)。本文的示例程序将应用State模式模拟奈德丽的技能释放。

# 示例需求分析

不知不觉中GoF的23种设计模式也快介绍到尾声了(19/23)，之前给出的设计模式的示例程序大多很简单，也很粗糙，其目的仅仅是为了说明某个设计模式的核心思想。本文将打破这一点(毕竟总写一些实验性质的小demo着实无聊)，会写一个相对完整的产品出来。关于奈德丽具体的英雄参数可参见[奈德丽数据介绍](http://cha.17173.com/lol/heros/details/76.html)。本程序会以奈德丽的满级(18级)属性为基础，再进行适当的提升(因为人物是会穿装备的)。

GUI设计技术为Java AWT，程序启动后初始状态如下：

![11.jpg](/images/blog_pic/Java 设计模式/19State模式/11.jpg)

这也是程序的主体面板界面。它的区域介绍如下：

![12.jpg](/images/blog_pic/Java 设计模式/19State模式/12.jpg)

默认形态为人形态。此时：

- **区域一：**人形态是有MP的，因此血条与蓝条均会展示，初始时默认血蓝全满。
- **区域二：**人形态下的立绘采用我最喜欢的枭姬皮肤(啦啦啦)。
- **区域三：**此时展示的是人形态下的技能图标。

当按下键盘上的R键后，即被认为释放了一次R技能(QWE技能同理)，此时奈德丽会从人形态切换为豹形态：

![13.jpg](/images/blog_pic/Java 设计模式/19State模式/13.jpg)

切换后：

- **区域一：**因为奈德丽在豹形态下是没有MP的，因此不绘制蓝条。
- **区域二：**立绘切换为枭姬皮肤的豹形态(没找到太合适的图，比较遗憾)。
- **区域三：**技能图标切换为豹形态。

关于技能图标，有以下几点需要说明：

第一，我们可以注意到上图中豹形态下的R技能"黑了一块"。这表示R技能在冷却。只有当它冷却完成后，我们才能再次释放，以切换为人形态。因为奈德丽的7个技能都是主动技能，因此其他技能的冷却机制同理。

当R技能冷却完成后：

![14.jpg](/images/blog_pic/Java 设计模式/19State模式/14.jpg)

我们可以注意到，相较于人形态，奈德丽的R技能的图标并没有改变。这也是我们要说的第二点，QWE技能是独立的，而R技能只有一个(因此我们才说奈德丽有7个技能，而非8个)。

QWE独立还意味着，QWE技能的冷却时间计算也是独立的，例如我们可以在人形态下快速释放QWE(俗称脸滚键盘)，使他们均进入冷却状态：

![15.jpg](/images/blog_pic/Java 设计模式/19State模式/15.jpg)

此时如果我们再按下R切换为豹形态，我们会发现豹形态下的QWE又是可释放的了，不过R技能因为人豹形态共用一个，因此进入了冷却状态。此时我们可以在豹形态下再QWE脸滚键盘一波：

![16.jpg](/images/blog_pic/Java 设计模式/19State模式/16.jpg)

当R技能冷却完成后，我们又可以切换回人形态，如果我们进行如上人-豹-人的操作的速度足够快，当我们再切回人形态时，情况可能是这样的：

![17.jpg](/images/blog_pic/Java 设计模式/19State模式/17.jpg)

R技能又开始接着计算冷却了，这个前文已经介绍过原因，并没有什么。但是除了冷却时间较短的Q技能之外，人形态下的WE技能并没有冷却完。这个效果当我们在R技能冷却好再次切到豹形态时也是同理的(只不过豹形态下技能CD普遍较短，不太容易看出来)。之所以会这样，是因为QWE技能独立同样意味着技能CD独立。举例来说，如果人形态的Q技能的CD为6秒，那么无论如何，它都只能6秒后才能再次释放，我们切换为豹形态会让我们看不到人形态Q技能的CD情况，但也仅仅只是看不到而已，它依然在不停的倒数自身的冷却时间。

由上文的技能CD介绍还能看出的是，人形态下释放QWE技能导致MP减少(蓝条变短)。而豹形态的技能则是无消耗的。R技能作为形态切换的桥梁也是无消耗的。这意味着，豹形态技能的释放只受CD的影响，而人形态在此之上，还需剩余的蓝量大于技能的消耗。

蓝条可以通过技能释放来减少。为了模拟血条减少，我们规定当按下键盘上的X键后，会扣一定数值的血量：

![18.jpg](/images/blog_pic/Java 设计模式/19State模式/18.jpg)

虽然用静态的图片无法表现，不过HP与MP均会按一定速率自动回复。

按照一般游戏的设定，蓝条始终都会是蓝色的，而血条则会在减少的过程中逐渐由绿转黄，再由黄转红(所谓的满血，黄血，红血)。本程序也是如此。上文已经展示了血量比较健康时血条的颜色，下面我们来看一张半血时的：

![19.jpg](/images/blog_pic/Java 设计模式/19State模式/19.jpg)

然后是残血时的：

![20.jpg](/images/blog_pic/Java 设计模式/19State模式/20.jpg)

奈德丽人形态下的E技能会为指向的英雄回复一定的HP，本程序默认会对自身释放，因此人形态释放E技能后会回复一定数值的HP(按照设定，回复量在一个范围之间，HP越低回复量越多)。除此之外，除了R技能会导致形态切换，其他技能(人形态的QW，豹形态的QWE)均为对敌方造成伤害，在本程序中不会有所体现。

当HP降到0时，英雄会死亡：

![21.jpg](/images/blog_pic/Java 设计模式/19State模式/21.jpg)

![22.jpg](/images/blog_pic/Java 设计模式/19State模式/22.jpg)

形态会被定格在死亡的那一刻。届时将不再接受任何键盘输入，也不会提供初始化的方法，只能关掉重来。此时：

- **区域一：**血条当然是清空了。而人形态下的蓝条将定格在死亡的时刻，不再变动。
- **区域二：**立绘将被虚化，同时添加表示阵亡的文字。
- **区域三：**技能图标将被虚化，同时清除可能会存在的尚未完成的CD遮挡。

呼~需求终于大致描述完了。累死我了。

作为一名技术，我们很少会关注产品经理的工作，不过即便是这么小的一个需求，我在整理并力求没有歧义的描述清楚(而且我并不敢说真的就描述清楚了)时，也费了很大的力气。这说明产品经理确实在日常的工作中为我们屏蔽了很多技术不愿意关注的点。之所以会有矛盾冲突，我想大多是因为需求本身就充斥着更多的冲突，没有产品经理，技术将更难以工作。

# 伪代码分析

需求描述完了，下面就来思考一下怎么写。

假设我们只考虑Q技能的释放，那么最容易想到的写法是这样的：

```
类 奈德丽 {

    方法 释放Q {
        if (当前状态为人形态) {
	    人形态下释放Q
	} else if (当前状态为豹形态) {
	    豹形态下释放Q
	} else {
	    异常
	}
    }
}
```

这样做当然没什么毛病，但是问题在于每当我们要做什么操作时都需要先判断一下状态。随着状态的增多，代码逻辑的复杂，这种判断的成本将越来越高，同时也越发的不易于维护。

如果使用State模式，我们就可以将形态抽象为类:

```
类 人形态 {

    方法 释放Q {
        人形态下释放Q
    }
}

类 豹形态 {

    方法 释放Q {
        豹形态下释放Q
    }
}
```

此时奈德丽类就可以这样写了：

```
类 奈德丽 {

    字段 当前状态

    方法 释放Q {
        当前状态.方法 释放Q
    }
}
```

显然，代码逻辑变得清晰了许多，也更利于维护。

# 示例代码

首先是类图：

![23.jpg](/images/blog_pic/Java 设计模式/19State模式/23.jpg)

因为希望能做出一个相对完整的产品，所以添加了很多无关State模式的代码。因此较之真实的程序，类图将只描述与State模式相关的那一部分。

本程序中的所有代码将被统一置于design19包下，结构如下：

![24.jpg](/images/blog_pic/Java 设计模式/19State模式/24.jpg)

下面将逐个贴出每个类的源码。

首先要说明的是，既然较之此前的设计模式(我想此后的其实也一样)，我费了更大的力气写需求分析。那么我在介绍示例代码时，也打算说得详细一些：并不仅仅是说State模式本身，而是把用到的其他的模式与想法也都详细介绍一下，算是我个人这段时间学习的总结。

程序是以简化版的MVC模式设计的(即只有M与V，没有C)。下面先来介绍M。

**Hero类**

```
package design19.model;

public abstract class Hero {

    protected boolean ifDeath;

    protected int maxHealth = 2 * (370 + 90 * 17);

    protected int healthRegen = (int)(3 * (1 + 0.12 * 17));

    protected int health = maxHealth;

    protected int ap;

    public abstract void useQ();

    public abstract void useW();

    public abstract void useE();

    public abstract void useR();

    public abstract boolean ifHaveMana();

    protected abstract double manaRate();

    public abstract String getImgKeyWord();

    public abstract double remainQ();

    public abstract double remainW();

    public abstract double remainE();

    public abstract double remainR();

    Hero() {
        // 回血
        new Thread() {
            @Override
            public void run() {
                while (true) {
                    if (Hero.this.ifDeath) {
                        Hero.this.health = 0;
                        break;
                    }
                    try {
                        int tempHealth = Hero.this.health + Hero.this.healthRegen;
                        Hero.this.health = tempHealth > Hero.this.maxHealth ? Hero.this.maxHealth : tempHealth;
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }.start();
    }

    public boolean isDeath() {
        return ifDeath;
    }

    public void damage(int value) {
        int tempHealth = this.health - value;
        if (tempHealth <= 0) {
            this.ifDeath = true;
            this.health = 0;
            return;
        }
        this.health = tempHealth;
    }

    public double getHealthRate() {
        return 1.0 * this.health / this.maxHealth;
    }

    public double getManaRate() {
        if (!this.ifHaveMana()) throw new UnsupportedOperationException();
        return this.manaRate();
    }

    long remainTime(Skill skill) {
        long pass = System.currentTimeMillis() - skill.last;
        return (pass - skill.cd) >= 0 ? 0 : (skill.cd - pass);
    }

    boolean check(String mark) {
        if (this.ifDeath) return false;
        if ("Q".equals(mark))
            if (this.remainQ() > 0) return false;
        if ("W".equals(mark))
            if (this.remainW() > 0) return false;
        if ("E".equals(mark))
            if (this.remainE() > 0) return false;
        if ("R".equals(mark))
            if (this.remainR() > 0) return false;
        return true;
    }
}
```

从逻辑上来说，Hero类与后文要介绍的Skill类是model模块中外部(本程序中是View)能感知到的仅有的两个类。因为英雄联盟有很多位英雄，对于外部调用者而言，针对不同英雄写不同的代码是很不现实的，相反，正如其他MVC中的V那样，V需要的是M能提供一套通用的API。具体到本示例，就是"英雄释放技能，造成影响"。这都是很抽象的东西，换句话说，如果我们将奈德丽换为另一位英雄，例如寒冰射手艾希，View层不应做任何或只应做极少的修改。

因为本示例的逻辑相对简单，因此对外暴露的API仅仅只有Hero类一个。Skill类依逻辑可以暴露API，只是没必要。而model中的其他类则是不准暴露。这点务必要理解透彻才可以。

为了不使得程序看着过于臃肿，除非是需要对外提供的API，model内部各个类之间互相使用字段时尽量都没有写get-set方法，而仅仅只是通过访问权限来屏蔽外界，这在比较严谨的场合(例如实际工作时的代码)往往是不可以的。

之所以将Hero声明为抽象类而非接口，比较直观的解释自然就是我们希望在其中添加一些属于实例的字段及方法，以及我们确信在本程序中继承Hero的Nidalee类不会有其他父类。不过更本质的原因在于，这种继承关系遵循合成聚合复用原则(奈德丽是一位英雄，即ISA)及里氏替换原则，这才是原因所在，最开始说的那个所谓的"直观的解释"，比起原因，更像是由原因导致的现象。

作为LOL这种竞技游戏通常都会有的属性，HP与MP。我只将HP及HP相关的属性作为字段加入到了Hero类中。其原因就在于所有的英雄都有HP，但并不是所有的英雄都会有MP。比如就会有德玛西亚之力盖伦，不祥之刃卡特琳娜这种完全没有MP的英雄，或是盲僧李青，蛮族之王泰达米尔这种拥有能量或是怒气等与MP性质类似的东西的英雄。再或者是甚至像本文介绍的拥有双形态的奈德丽，某个形态下像大多英雄那样拥有MP，某个形态下又像盖伦那样无消耗。基于这个考虑，我将MP沉到了再下一个层级：如果这个英雄有MP，那么就在自己内部添加。而Hero这一层尽量只添加所有英雄都会有的字段。这也是抽象的本质思路所在：抽取相同的部分以形成更高的层级。因为相同的原因，就很容易理解Hero类中剩下的两个字段了：表示是否死亡的ifDeath，以及表示魔法能力的ap值。作为一个英雄，其实还会有更多复杂的属性，例如ad，护甲，魔抗，移速等等。本示例只使用了少量需要的部分。

除了字段之外，我们还将其他对HP的操作都尽量提到了Hero这一层，因为这是所有英雄通用的。我在Hero的构造函数中起了一个线程模拟回血。还提供了damage()方法供外部调用模拟扣血。

use Q/w/E/R方法表示外界按下了一次QWER键。而remain Q/w/E/R方法则返回QWER技能技能CD恢复的百分比。很显然，该值的取值范围为[0,1]。当该值等于0时说明技能没有进入冷却，可以释放(当然仅仅只是冷却好了，人形态下还要蓝够才可以)；而等于1则说明技能刚进入冷却。

ifHaveMana()这个方法会判断英雄是否有蓝条(本程序在设计时就不考虑怒气能量等奇怪的东西了)。依上文所说的MP下沉的思路，这个方法放在Hero类里虽然有点不太美观，但终归是还能接受。而getManaRate()方法就有些诡异了，它返回的是当前MP值占MP最大值的比例，是为了显示蓝条长度用的。由代码不难看出，当外部调用该方法后，会先再判断一次ifHaveMana()，如果返回false，即英雄没有MP，则抛出异常。反之才会调用子类本身计算比例的manaRate()方法，这里运用了Template Method模式。不过这不是重点。我想说的是，对于没有MP的英雄而言，这个方法是根本不该被调用的，这种提供了功能却在内部抛出异常的做法相当于甩锅给调用方，显然是很不优雅的。更有问题的是，明明将MP下沉到了更下一层，却还是在Hero这一层出现了这么具体的计算MP相关值的方法，看着着实扎眼。说了这么多，之所以千不该万不是还是把代码写成了这样，是因为我们在写代码时，比起面面俱到，更多时候是在两害相权取其轻。因为我们要向外界暴露一个通用的Hero而非具体的英雄，那么在外界需要蓝条时，只能做一个这样折中的方案。或是将程序写得更复杂：例如抽象出一个能统合MP，怒气，能量，甚至是无的东西，代表释放技能的代价。不过本程序显然没必要那么复杂就是了。

**Skill类**

```
package design19.model;

public class Skill {

    long cd;

    long last;

    int cost;

    Skill(long cd, int cost) {
        this.cd = cd;
        this.cost = cost;
    }
}
```

相对而言，Skill类就要简单得多了。也没有提供对外的API。

**Nidalee类**

```
package design19.model;

public class Nidalee extends Hero {

    Skill skillR = new Skill(4000L, 0);

    NidaleeState state;

    public Nidalee() {
        NidaleeState humanState = new HumanState(this);
        NidaleeState leopardState = new LeopardState(this);
        humanState.otherState = leopardState;
        leopardState.otherState = humanState;
        this.state = humanState;
    }

    @Override
    public void useQ() {
        this.state.useQ();
    }

    @Override
    public void useW() {
        this.state.useW();
    }

    @Override
    public void useE() {
        this.state.useE();
    }

    @Override
    public void useR() {
        this.state.useR();
    }

    @Override
    public boolean ifHaveMana() {
        return this.state.maxMana > 0;
    }

    @Override
    public double manaRate() {
        return 1.0 * this.state.mana / this.state.maxMana;
    }

    @Override
    public String getImgKeyWord() {
        return this.state.type;
    }

    @Override
    public double remainQ() {
        if (this.ifDeath) return 0.0;
        return 1.0 * this.remainTime(this.state.skillQ) / this.state.skillQ.cd;
    }

    @Override
    public double remainW() {
        if (this.ifDeath) return 0.0;
        return 1.0 * this.remainTime(this.state.skillW) / this.state.skillW.cd;
    }

    @Override
    public double remainE() {
        if (this.ifDeath) return 0.0;
        return 1.0 * this.remainTime(this.state.skillE) / this.state.skillE.cd;
    }

    @Override
    public double remainR() {
        if (this.ifDeath) return 0.0;
        return 1.0 * this.remainTime(this.skillR) / this.skillR.cd;
    }
}
```

state字段表示奈德丽当前处于的状态。在构造函数中，我们将默认的状态设置为人形态。然后我们又声明了一个豹形态的对象。将人形态与豹形态互相绑定。这意味着，在按下R技能，需要进行形态切换时，Nidalee类是不需要做任何操作的，切换操作将由表示形态的类完成：人形态时变为豹形态，豹形态时变为人形态。进而，Nidalee类在使用state时(我们可以看到，Nidalee类中绝大多数的操作最终实际都是由state完成的)，并不关心当前到底处于什么形态，它都会当做只有一个形态那样去使用。

本程序中，Nidalee类中只会存储一个表示形态的字段。形态的切换由形态本身完成。它并不能统计出当前一共有多少种形态 -- 这当然是State模式的设计方式之一。这样的好处在于既然Nidalee类将状态相关的处理都委托给各State类去做，那么"状态切换"显然也是相关处理之一。那么在它被触发的地方直接完成显然是最简洁干脆的做法 -- 正如本文所做的那样。不过，这样做的问题在于，State之间需要彼此感知，因为唯有这样，一个State才能知道要切换为谁。这其实并不是十分符合逻辑：因为各State类之间其实是兄弟关系，并没有义务互相感知。从这个思路来想，负责切换的工作应该交给再上一层，也就是这些State类的宿主Nidalee类。此时，Nidalee类就需要记录自己保管的所有State类，以及它们之间的切换逻辑。

奈德丽共有7个不同的技能。我们将人形态的QWE与豹形态的QWE均放在了对应形态的类中，因为我们认为这些技能是属于那个形态特有的。而将人豹共用的R技能提到了Nidalee类中。同理，我们依然没有将MP放到Nidalee类中，而是将它继续下放：因为只有人形态下才有MP，豹形态下没有。

**NidaleeState类**

```
package design19.model;

public abstract class NidaleeState {

    protected Nidalee nidalee;

    protected String type;

    protected NidaleeState otherState;

    protected int maxMana;

    protected int manaRegen;

    protected int mana;

    protected Skill skillQ;

    protected Skill skillW;

    protected Skill skillE;

    protected abstract void handleQ();

    protected abstract void handleW();

    protected abstract void handleE();

    NidaleeState(Nidalee nidalee) {
        this.nidalee = nidalee;
    }

    void useQ() {
        if (!this.nidalee.check("Q")) return;
        if (this.mana < this.skillQ.cost) return;
        this.skillQ.last = System.currentTimeMillis();
        this.handleQ();
    }

    void useW() {
        if (!this.nidalee.check("W")) return;
        if (this.mana < this.skillW.cost) return;
        this.skillW.last = System.currentTimeMillis();
        this.handleW();
    }

    void useE() {
        if (!this.nidalee.check("E")) return;
        if (this.mana < this.skillE.cost) return;
        this.skillE.last = System.currentTimeMillis();
        this.handleE();
    }

    void useR() {
        if (!this.nidalee.check("R")) return;
        this.nidalee.skillR.last = System.currentTimeMillis();
        this.nidalee.state = this.otherState;
    }
}
```

NidaleeState类是奈德丽这个英雄特有的，由其双状态提取共性后抽象出的类。正如V需要一个相对通用的Hero那样。Nidalee类也需要一个通用的表示状态的类。对于更上面的层级而言(Hero，View)，奈德丽拥有双形态是其自身内部的事，它们并不会，也不需要感知得到。

另外一个有趣的点是，Hero作为直接暴露在外的类，会提供很多与外部逻辑紧密相关的功能。例如Hero中的ifHaveMana()，而Nidalee作为它的子类也必须实现。但是作为最终工作者的NidaleeState中却没有这个方法 -- Nidalee的ifHaveMana()实际是通过判断当前状态的maxMana来实现的。关于这一点我想说的是，外部需求是在不停扩充的，需求也将千变万化。而不论Hero也好，Nidalee也罢，其实都是model模块内部的类。这会导致model与外部需求耦合得过于紧密，不利于代码的维护。而通常的做法是，我们会在M与V之间再加入一个DAO，也就是数据传输层。因为本质上来说，MVC其实就是"V向M要数据"以及"V触发M修改自身的数据"(很少会出现"M主动向V推送数据"的情况，因为下层一般无需对上层负责，毕竟M都是被动的，笑)。而DAO则是M与V之间的桥梁。当引入DAO后，这部分与V耦合得过于紧密的代码就可以自M转移到DAO中了。从更大的视角来看，这个所谓的DAO，实际上是MVC中C所承载的功能的一部分。所以说，出来混终归是要还的。本示例程序节省了C这一层，终究还是给程序引入了风险(话是这么说没错啦，小程序其实问题不大。还是要具体问题具体分析)。

**HumanState类**

```
package design19.model;

public class HumanState extends NidaleeState {

    HumanState(Nidalee nidalee) {
        super(nidalee);
        this.type = "human";
        this.maxMana = 4 * (220 + 45 * 17);
        this.manaRegen = (int)(5 * (0.9 + 0.06 * 17));
        this.mana = maxMana;
        this.skillQ = new Skill(6000L, 90);
        this.skillW = new Skill(18000L, 60);
        this.skillE = new Skill(10000L, 120);
        // 回蓝
        new Thread() {
            @Override
            public void run() {
                while (true) {
                    if (HumanState.this.nidalee.ifDeath) break;
                    try {
                        int tempMana = HumanState.this.mana + HumanState.this.manaRegen;
                        HumanState.this.mana = tempMana > HumanState.this.maxMana ? HumanState.this.maxMana : tempMana;
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }.start();
    }

    @Override
    protected void handleQ() {
        this.modMana(this.skillQ.cost);
    }

    @Override
    protected void handleW() {
        this.modMana(this.skillW.cost);
    }

    @Override
    protected void handleE() {
        this.modMana(this.skillE.cost);
        double loseHealthRate = 1.0 - this.nidalee.getHealthRate();
        double addHealthBase = 115 + (230 - 115) * loseHealthRate;
        double addHealthAP = this.nidalee.ap * (0.325 + (0.65 - 0.325) * loseHealthRate);
        int tempHealth = this.nidalee.health + (int)(addHealthBase + addHealthAP);
        this.nidalee.health = tempHealth >= this.nidalee.maxHealth ? this.nidalee.maxHealth : tempHealth;
    }

    private void modMana(int cost) {
        if (this.mana < cost) return;
        int tempMana = this.mana - cost;
        this.mana = tempMana > 0 ? tempMana : 0;
    }
}
```

奈德丽的人形态。我们可以看到，MP相关的操作都在这个类中。

**LeopardState类**

```
package design19.model;

public class LeopardState extends NidaleeState {

    LeopardState(Nidalee nidalee) {
        super(nidalee);
        this.type = "leopard";
        this.skillQ = new Skill(5000L, 0);
        this.skillW = new Skill(3500L, 0);
        this.skillE = new Skill(6000L, 0);
    }

    @Override
    protected void handleQ() {}

    @Override
    protected void handleW() {}

    @Override
    protected void handleE() {}
}
```

handle Q/W/E方法都是空的，这意味着这些技能不会产生什么效果。实际中当然不会这样，因此这样写程序其实并没有问题。

介绍完了HumanState与LeopardState，我想说的是，很多时候，表征状态的类通常都是单例的，因为状态本就是一个相对抽象的东西。只不过，在本程序中，状态还和很多特有的属性(例如MP，技能恢复CD)挂钩，因此不能声明为单例。

**View类**

```
package design19.view;

import java.awt.Color;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.awt.image.BufferedImage;
import java.io.IOException;

import javax.imageio.ImageIO;

import design19.model.Hero;
import design19.model.Nidalee;

public class View extends Frame {

    private static final long serialVersionUID = 1L;

    private Hero nidalee = new Nidalee();

    public void launchFrame() {
        super.setLocation(500, 200);
        super.setSize(330, 713);
        new Thread(this.new RepaintRunnable()).start();
        super.addWindowListener(
            new WindowAdapter() {
                @Override
                public void windowClosing(WindowEvent e) {
                    System.exit(0);
                }
            }
        );
        super.addKeyListener(
            new KeyAdapter() {
                @Override
                public void keyReleased(KeyEvent e) {
                    switch(e.getKeyCode()) {
                    case KeyEvent.VK_Q:
                        View.this.nidalee.useQ();
                        break;
                    case KeyEvent.VK_W:
                        View.this.nidalee.useW();
                        break;
                    case KeyEvent.VK_E:
                        View.this.nidalee.useE();
                        break;
                    case KeyEvent.VK_R:
                        View.this.nidalee.useR();
                        break;
                    case KeyEvent.VK_X:
                        View.this.nidalee.damage(1000);
                        break;
                    }
                }
            }
        );
        super.setVisible(true);
    }

    @Override
    public void paint(Graphics g) {
        // 基本参数
        int leftBar = 11;
        int aboveBar = 45;
        int imgWidth = 308;
        int imgHeight = 560;
        int skillWidth = 308 / 4;
        // 血条蓝条
        int slotHeight = 10;
        g.setColor(this.getHealthColor());
        g.fillRect(leftBar, aboveBar, (int)(imgWidth * this.nidalee.getHealthRate()), slotHeight);
        if (this.nidalee.ifHaveMana()) {
            g.setColor(Color.BLUE);
            g.fillRect(leftBar, aboveBar + slotHeight, (int)(imgWidth * this.nidalee.getManaRate()), slotHeight);
        }
        // 图片
        g.drawImage(this.loadImgWeaken(this.nidalee.getImgKeyWord() + ".jpg"), leftBar, aboveBar + 2 * slotHeight, null);
        // 技能图标
        int skillH = aboveBar + 2 * slotHeight + imgHeight;
        g.drawImage(this.loadImgWeaken(this.nidalee.getImgKeyWord() + "Q.jpg"), leftBar, skillH, null);
        g.drawImage(this.loadImgWeaken(this.nidalee.getImgKeyWord() + "W.jpg"), leftBar + skillWidth, skillH, null);
        g.drawImage(this.loadImgWeaken(this.nidalee.getImgKeyWord() + "E.jpg"), leftBar + skillWidth * 2, skillH, null);
        g.drawImage(this.loadImgWeaken("R.jpg"), leftBar + skillWidth * 3, skillH, null);
        // 技能冷却
        g.setColor(Color.BLACK);
        int qCDHeight = (int)(this.nidalee.remainQ() * skillWidth);
        g.fillRect(leftBar, skillH + skillWidth - qCDHeight, skillWidth, qCDHeight);
        int wCDHeight = (int)(this.nidalee.remainW() * skillWidth);
        g.fillRect(leftBar + skillWidth, skillH + skillWidth - wCDHeight, skillWidth, wCDHeight);
        int eCDHeight = (int)(this.nidalee.remainE() * skillWidth);
        g.fillRect(leftBar + skillWidth * 2, skillH + skillWidth - eCDHeight, skillWidth, eCDHeight);
        int rCDHeight = (int)(this.nidalee.remainR() * skillWidth);
        g.fillRect(leftBar + skillWidth * 3, skillH + skillWidth - rCDHeight, skillWidth, rCDHeight);
        // 阵亡文字
        if (this.nidalee.isDeath()) g.drawImage(this.loadImg("death.png"), leftBar + 35, aboveBar + 250, null);
    }

    private class RepaintRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    View.this.repaint();
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

    private BufferedImage loadImg(String name) {
        BufferedImage bImage = null;
        try {
            bImage = ImageIO.read(View.class.getClassLoader().getResource("design19/img/" + name));
        } catch (IOException e) {
            e.printStackTrace();
        }
        return bImage;
    }

    private BufferedImage loadImgWeaken(String name) {
        BufferedImage bufferedImage = this.loadImg(name);
        if (!this.nidalee.isDeath()) return bufferedImage;
        final int width = 1;
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
        return bufferedImage;
    }

    private Color getHealthColor() {
        int red;
        int green;
        if (this.nidalee.getHealthRate() >= 0.5) {
            green = 255;
            red = (int)(255 * (1.0 - this.nidalee.getHealthRate()) * 2);
        } else {
            red = 255;
            green = (int)(255 * this.nidalee.getHealthRate() * 2);
        }
        return new Color(red, green, 0);
    }
}
```

代码很长，不过其中大多是Java AWT套路话的东西，需要说的只有以下几点。

首先，为了代码书写简单，我写了很多硬编码。通常这种涉及面板参数的操作，都会写得更灵活，例如支持面板整体的缩放等。其次，在加载图片时，我其实是每刷新一次面板就重新加载了一次，这显然是很浪费CPU的，通常来说都会在一个地方将需要用的图片加载入内存，形成静态字段。

其次，就是两个小算法啦。先来说第一个：血条绿-黄-红的渐变。这个功能主要写在getHealthColor()方法中。本质上来说，这个算法就是通过控制RGB中RED-GREEN-BLUE三个分量的值来调制出需要的颜色。对于RGB颜色而言：

- 绿：(0,255,0)
- 黄：(255,255,0)
- 红：(255,0,0)

假如我们将HP的变化趋势考虑为递减(递增其实也同理，相当于是逆过程)：

首先，无论如何，蓝色分量均为0。

在HP由100%-50%的过程中，绿色分量始终为255，而红色分量由0逐渐增加至255。

当血量达到50%时，RGB颜色为(255,255,0)，即标准的黄色。

随后，在HP由50%-0的过程中，红色分量始终为255，而绿色分量则由255逐渐减少为0。

然后再来说第二个：也就是图片虚化效果的实现。功能代码在loadImgWeaken()方法中，实现的效果就是行与列均是每隔一个像素点输出一个，形成网格效果。同时，由于输出的像素点变少，图片整体也会变暗。具体的算法逻辑可参见[Java AWT-以像素为单位操纵图片](/2018/05/20/Java AWT-以像素为单位操纵图片/)。

**Main类**

```
package design19;

import design19.view.View;

public class Main {

    public static void main(String[] args) {
        new View().launchFrame();
    }
}
```

运行后，即可按需求输出结果。

**用到的图片**

最后，贴一下本程序用到的图片。

人形态立绘human.jpg

![25.jpg](/images/blog_pic/Java 设计模式/19State模式/25.jpg)

---

豹形态立绘leopard.jpg

![26.jpg](/images/blog_pic/Java 设计模式/19State模式/26.jpg)

---

人形态Q技能humanQ.jpg

![27.jpg](/images/blog_pic/Java 设计模式/19State模式/27.jpg)

---

人形态W技能humanW.jpg

![28.jpg](/images/blog_pic/Java 设计模式/19State模式/28.jpg)

---

人形态E技能humanE.jpg

![29.jpg](/images/blog_pic/Java 设计模式/19State模式/29.jpg)

---

豹形态Q技能leopardQ.jpg

![30.jpg](/images/blog_pic/Java 设计模式/19State模式/30.jpg)

---

豹形态W技能leopardW.jpg

![31.jpg](/images/blog_pic/Java 设计模式/19State模式/31.jpg)

---

豹形态E技能leopardE.jpg

![32.jpg](/images/blog_pic/Java 设计模式/19State模式/32.jpg)

---

R技能R.jpg

![33.jpg](/images/blog_pic/Java 设计模式/19State模式/33.jpg)

---

死亡文字提示death.png

![34.png](/images/blog_pic/Java 设计模式/19State模式/34.png)

# 登场角色

上面的示例程序介绍了State模式的Java实现，下面咱们试着跳出语言层面，抽象出State模式中登场的角色。

**State(状态)**

在示例程序中，由NidaleeState类扮演这个角色。

**ConcreteState(具体的状态)**

在示例程序中，由HumanState及LeopardState联袂扮演这个角色。

**Context(上下文)**

实际就是拥有State的宿主，在示例程序中，由Nidalee类扮演这个角色。

下面是抽象后，无关语言的类图：

![35.jpg](/images/blog_pic/Java 设计模式/19State模式/35.jpg)

较之示例程序，本类图少了不少类。其原因就在于，从本质上来说。State模式就是在将State角色自Context角色中分离出去。因此核心角色其实就只有这两个。

此外，示例程序中的NidaleeState是一个抽象类，而本类图中的State是一个接口。之所以会产生这种不同，是因为State只要表示的含义是一个抽象的状态即可，具体的形式应依具体使用场景而定。

# 易于增加xx，难以xx

在State模式中，增加一个新的State相对容易，我们只需要让它实现上层的抽象类或接口，然后按约束编写代码即可。而在上层的抽象类或接口中增加一个新的方法则相对费事：这需要每一个State都增加这个新的方法。因为代码相对分散，每个State的编码思路可能大不相同，所以写起来会麻烦得多。

仔细回忆此前写过的设计模式相关的博客，这种"易于xx，难以xx"其实已经出现过很多次了。例如，在[8.Abstract Factory模式]()中我们就曾说过，Abstract Factory模式"易于扩展具体的工厂，难以增加新的零件"。

其实这种特性源于继承或是实现。按照抽象的上层的约束写成新的下层相对容易，因为这遵循开闭原则。此时只需增加新代码，而无需对原有代码做出修改。而在抽象的上层中添加新功能则相对困难，因为这相当于所有已有的下层类都需要修改，违背了开闭原则。