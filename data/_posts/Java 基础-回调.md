---
title: Java 基础-回调
date: 2017-12-02 11:36:49
tags: [Java,回调]
categories: Java 基础
---

本文将以两个小例子来说明钩子(Hook)及回调(CallBack)。示例中的人物均出自[东方Project](https://baike.baidu.com/item/%E4%B8%9C%E6%96%B9Project/6217040?fr=aladdin)。

<!-- more -->

# 例子1：钩子

魔理沙(Marisa)是一个乐于钻研魔法，研发魔法道具的好孩子，最初，她研发魔法道具的套路为：

```
public class Marisa {

    public void research() {
        System.out.println("收集蘑菇等制作道具的素材");
        System.out.println("调戏灵梦");
        System.out.println("查找素材相关的魔法理论知识");
        System.out.println("动手开始研发");
        System.out.println("造出道具");
    }
}
```

让魔理沙比较苦手的是第三步"查找素材相关的魔法理论知识"，为此她不得不时常去找自己的后宫帕秋莉(Patchouli)咨询魔法基础姿势，于是逐渐的，研发流程变成了如下的样子：

```
public class Marisa {

    public void research(Patchouli patchouli) {
        System.out.println("收集蘑菇等制作道具的素材");
        System.out.println("调戏灵梦");
        patchouli.teach();
        System.out.println("动手开始研发");
        System.out.println("造出道具");
    }
}

class Patchouli {

    void teach() {
        System.out.println("边咳嗽边传授素材相关的魔法理论知识");
    }
}
```

这样魔理沙不仅仅能学习到更多的膜法姿势，还加深了与姬友的感情。然而我们的姆Q实在是太病弱了，久而久之身体逐渐就吃不消了。于是心疼的魔理沙有问题时便不仅仅去找帕秋莉，也会去找她的另一个姬友爱丽丝·玛格特罗依德(Alice)，当她去找小爱时，流程是这样的：

```
package com.test;

public class Marisa {

    public void research(Alice alice) {
        System.out.println("收集蘑菇等制作道具的素材");
        System.out.println("调戏灵梦");
        alice.teach();
        System.out.println("动手开始研发");
        System.out.println("造出道具");
    }
}

class Patchouli {

    void teach() {
        System.out.println("边咳嗽边传授素材相关的魔法理论知识");
    }
}

class Alice {

    void teach() {
        System.out.println("边傲娇边传授素材相关的魔法理论知识");
    }
}
```

**至此钩子(Hook)的基本样式已形成，多数代码如同模版一样不变，而只有少数代码需调用外部，从而实现了功能的特化和分离。这种思想源自23种Java设计模式中的模版方法模式。不过此时依然不够灵活，尚有改进空间。**

这样又度过了一段神仙般的日子，不过，魔理沙也愈发觉得自己在后宫关系的处理上无法做到一碗水端平了。她总是不经意的思考："最近是不是找帕秋莉的次数有点多，小爱会不会吃醋砍我啊"，"上次和小爱玩蘑菇的时候被射命丸文那个狗仔拍到了，她万一告诉帕秋莉怎么办啊"凡此等等。并为此烦恼不止。突然有一天魔理沙灵光乍现，醒悟了一件事：我这是在找我的好姬友探讨魔法姿势啊，又不是什么见不得光的事，在这件事上她们二人都是我的良师益友，为什么要分得这么清楚呢？解开心结后的魔理沙舒畅无比，她心目中的流程变成了这样：

```
public class Marisa {

    public void research(Teacher teacher) {
        System.out.println("收集蘑菇等制作道具的素材");
        System.out.println("调戏灵梦");
        teacher.teach();
        System.out.println("动手开始研发");
        System.out.println("造出道具");
    }
}

interface Teacher {

    void teach();
}

class Patchouli implements Teacher {

    @Override
    public void teach() {
        System.out.println("边咳嗽边传授素材相关的魔法理论知识");
        
    }
}

class Alice implements Teacher {

    @Override
    public void teach() {
        System.out.println("边傲娇边传授素材相关的魔法理论知识");
    }
}
```

**至此，便完成了一个可灵活扩展配置的钩子。**

# 例子2：回调

某一天，上白泽慧音(Keine)老师给橙(Chen)留了一个家庭作业，要她计算1+1等于几。于是橙当晚回家便自己算了起来：

```
public class Chen {

    private int add(int i, int j) {
        return i + j;
    }

    public void homeWork(int i, int j) {
        System.out.println("完成作业，结果为:" + this.add(i, j));
    }
}
```

**首先调用homeWork方法，其内部又触发add方法，整个操作都在Chen这个类的内部完成，不涉及回调**

慧音老师看后很是满意，于是第二天留的家庭作业便稍难了一些，变为计算12+23等于几。

这就超出橙的知识范围了，因为她只会计算十以内的加法。没办法，她只好找到了家长八云蓝(Ran)。作为全幻想乡最强大的式神，蓝拥有着超凡的数学天赋和计算能力，比如她可以快速的口算出两个庞大整数的加法结果：

```
public class Ran {

    public int add(int i, int j) {
        return i + j;
    }
}
```

溺爱孩子的蓝挨不住橙的软磨硬泡，只好算出结果后告诉橙，然后橙再将结果写到作业本上完成了作业：

```
public class Ran {

    public int add(int i, int j) {
        return i + j;
    }
}

class Chen {

    public void homeWork(Ran ran, int i, int j) {
        System.out.println("完成作业，结果为:" + ran.add(i, j));
    }
}
```

**此时任务已不完全由Chen完成了，但依然不是回调**

没想到第三天，慧音老师出了一道更难的题，竟然已经达到了恐怖的3位数加法：123+456。

至此橙连作业本都不想碰了，她求蓝直接帮她写作业得了。在橙强大的卖萌攻势下，最终蓝还是屈服了，于是当晚的情况是这样的：

```
public class Ran {

    private int add(int i, int j) {
        return i + j;
    }

    public void addAgent(int i, int j, Chen chen) {
        chen.homeWork(this.add(i, j));
    }
}

class Chen {

    public void homeWorkAgent(Ran ran, int i, int j) {
        ran.addAgent(i, j, this);
    }

    public void homeWork(int result) {
        System.out.println("完成作业，结果为:" + result);
    }
}
```

**至此回调基本成型：首先调用Chen.homeWorkAgent方法，其内部向Ran.addAgent求助。此时对于Chen而言，整个作业的事算是交代完了，她无需再操心到底是加是减，到底要写什么答案。而对于Ran而言，其在算出答案后，还需要回过头来再调回Chen中的方法homeWork(这也是回调名称的由来)，最终由Ran完成作业。**

这样又过了很多天之后，橙的同班同学琪露诺(Cirno)听说了这件事并羡慕不以。毕竟她只能计算5以内的加法。于是她也委托橙让蓝帮帮自己。当晚橙回家又是一通疯狂卖萌，蓝没有办法，只能边擦着鼻血边答应了下来。只是在帮琪露诺时，要做一个小小的角色转换：

```
public class Ran {

    private int add(int i, int j) {
        return i + j;
    }

    public void addAgent(int i, int j, Cirno cirno) {
        cirno.homeWork(this.add(i, j));
    }
}

class Cirno {
    
    public void homeWorkAgent(Ran ran, int i, int j) {
        ran.addAgent(i, j, this);
    }
    
    public void homeWork(int result) {
        System.out.println("完成作业，结果为:" + result);
    }
}
```

如是反复数日后，蓝也不禁觉得有些麻烦。然后她突然想到：虽然琪露诺这个傻9完全没法和自家的小宝贝相比，然而在做题这件事上二人的身份其实是一样的，为什么要分得这么清楚呢？想通之后顿觉豁然开朗，于是此后的流程就变成了这样：

```
public class Ran {

    private int add(int i, int j) {
        return i + j;
    }

    public void addAgent(int i, int j, HomeWorkForHelp homeWorkForHelp) {
        homeWorkForHelp.homeWork(this.add(i, j));
    }
}

interface HomeWorkForHelp {

    public void homeWork(int result);
}

class Chen implements HomeWorkForHelp {

    public void homeWorkAgent(Ran ran, int i, int j) {
        ran.addAgent(i, j, this);
    }

    @Override
    public void homeWork(int result) {
        System.out.println("完成作业，结果为:" + result);
        
    }
}

class Cirno implements HomeWorkForHelp {

    public void homeWorkAgent(Ran ran, int i, int j) {
        ran.addAgent(i, j, this);
    }

    @Override
    public void homeWork(int result) {
        System.out.println("完成作业，结果为:" + result);
        
    }
}
```

**至此便完成了一个灵活的回调**

# 总结

结合以上两个小例子，让我们来总结下钩子与回调的异同。

首先，两个例子的基本套路是一样的。都有两大阵营：寻求帮助的人和提供帮助的人。我们不妨分别命名为A与B。

不过，在钩子的例子中，故事的主人公(或者说主视角)为A，即雾雨魔理沙，只有一个。而B的角色却有复数个：帕秋莉及爱丽丝。而在回调的例子中，故事的主人公(或者说主视角)则是B，即为八云蓝，只有一个。而A的角色却有复数个：橙和琪露诺。

另一个不同之处在于，在回调中，B必须再次调回A。换句话说，这其实是方法调用过程中同步及异步的差异(可参见[Java 并发-同步异步阻塞非阻塞](/2017/10/03/Java 并发-同步异步阻塞非阻塞/)):

- 钩子：A调用B后等待，待B那边的结果返回后继续执行。即采取的是同步策略。

- 回调：A调用B后就不管了，待B计算出结果后需要再通知A，此时A根据结果再采取行动。即采取的是异步策略。

需要注意的是，本文中关于回调的代码其实并不是真正异步的，因为A的homeWorkAgent方法实际上只有当B的addAgent方法完成时才能完成，从代码执行逻辑上依然是同步的。要想实现真的异步，还是要采用真正并发编程的策略，让B的addAgent方法完全是在另一个新的线程中执行才可以。同时，如果单看示例中的代码逻辑，橙其实也并非是将作业完全委派给了蓝，因为毕竟homeWork方法还是橙自己的，只是触发人变为了蓝，细追究的话最终干活的人还是橙自己。不过作为一个例子，我想这样已经足以说明回调的原理了。

最后我想说的是，在两个小例子中，我故意用了类似的话：

**突然有一天魔理沙灵光乍现，醒悟了一件事：我这是在找我的好姬友探讨魔法姿势啊，又不是什么见不得光的事，在这件事上她们二人都是我的良师益友，为什么要分得这么清楚呢？解开心结后的魔理沙舒畅无比**

**虽然琪露诺这个傻9完全没法和自家的小宝贝相比，然而在做题这件事上二人的身份其实是一样的，为什么要分得这么清楚呢？想通之后顿觉豁然开朗**

这也是我们在解决多角色问题时通用的思路：为什么要分得那么清楚呢？不要过多的被对象复杂的身份所干扰，我们所须关注的仅仅是针对当前功能它们所扮演的是什么角色，只要一样，我们就可以将其抽象为一类，进而提高代码的灵活性。实际上，这就是面向接口编程的精髓所在。