---
title: Java 基础-碎片
date: 2017-09-27 14:31:49
tags: [Java]
categories: Java 基础
---

# 面向对象的特征

<!-- more -->

面向过程的程序是由事件驱动的。即先发生事件1，再发生事件2，然后是事件3...直至程序结束。面向过程中的过程即为事件依序执行的过程。定义每一个事件执行行为的规则称为"算法"，事件的核心意义在于对"数据"的处理：事件导致数据的值发生了变化。

面向对象的程序的核心为对象。面向过程中的"数据"变成了对象的"属性"，面向过程中的"算法"变成了对象的"行为"。即对象1做出了行为1，对象2做出了行为2...各对象的行为影响了自己或他人的属性，直至程序结束。

举个小例子：小王打了小李。面向过程关注的是"打人"这件事，具体到事件内部，打人者为小王，被打者为小李。面向对象关注的是小王和小李这两个人，小王做出了打小李的行为。很显然，面向过程及面向对象都能准确的描述这件事，只是思考的角度不同。

**封装**

- 面向过程的核心为"算法处理数据"，算法与数据均为最顶级的概念，即没有一个概念能让这二者从属于它。面向对象的核心为"对象"。一切都是对象，对象就是一切。对象是最顶级的概念，属性及行为均从属于对象。换句话说，对象封装了属性及行为。
- 对象就像一个黑盒。优秀的代码会隐藏一切可隐藏的属性或行为而只暴露必要的对外可见的部分。

**继承**

类似于生物学分类中的"界门纲目科属种"， 子类/派生类 继承 父类/超类/基类 ，表明子类默认拥有父类的属性及行为。同时如果有必要，子类也可以通过重写父类的属性及行为的方式展现自身不同于父类的个性。

**多态**

多态是指同样的静态类型因实际类型的不同在调用同一个方法时做出了不同的行为。实质上，多态是动态单分派的体现，其核心为重写。关于重写，详见[Java基础-重载与重写](/2017/10/10/Java基础-重载与重写/)。

**抽象**

面向对象的核心特征只有封装继承多态，抽象是一个不那么"特"的特征。因此有时并不会算上抽象。

抽取各具特色的对象之间的共性形成类这一模版的过程称为抽象。因为对象内封装了属性及行为，因此抽象也可细分为数据抽象及行为抽象。

# 访问修饰符

![0.jpg](/images/blog_pic/Java 基础/碎片/0.jpg)

无访问修饰符的情况也称其修饰符为 default/friendly 。

# Math.round(double a)的原理

round表示就近取整。实现原理为a加上0.5后再下取整(不大于传入值的最大整数)。

Math.round(-11.7)：-11.2下取整，返回-12。

Math.round(-11.2)：-10.7下取整，返回-11。

Math.round(11.7)：12.2下取整，返回12。

Math.round(11.2)：11.7下取整，返回11。

# switch的可作用类型

[JDK1.1,JDK1.5)时，switch(expr)的expr只能是char,byte,short,int。其实对于switch而言，能接收的仅仅就只有int。char,byte,short在传入时会发生自动类型转换被转为int。

JDK1.5新定义了枚举类型。从JDK1.5开始，expr也可以是enum类型。

从JDK1.7开始，expr还可以是字符串。

小例子：

```
public class Test {

    public static void main(String[] args) {
        String str = "八云紫";
        switch (str) {
            case "雾雨魔理沙":
                System.out.println("此为少女");
                break;
            case "八云紫":
                System.out.println("此为大妈");
                break;
            default:
                System.out.println("就当是少女吧");
        }
    }
}
```

输出：

```
此为大妈
```

到JDK1.7为止，long始终不可用于switch。

# 利用switch case穿透的小技巧

```
public class Test {
    public static void main(String[] args) {
        char c = (char)('a' + (int)(26 * Math.random()));
        System.out.print(c + "是");
        switch (c) {
            case 'a':
            case 'e':
            case 'i':
            case 'o':
            case 'u':
                System.out.println("元音");
                break;
            case 'y':
            case 'w':
                System.out.println("半元音");
                break;
            default:
                System.out.println("辅音");
        }
    }
}
```

# Java有哪些数据类型？

- 基本数据类型(primitive type) --> 详见[Java 基础-基本数据类型](/2017/09/27/Java 基础-基本数据类型/)
- 引用类型(reference type)
- 枚举类型(enumeration type)

# 行内注释

可在行内只修饰一个变量：

```
String /*其实是老婆婆*/bayun = "17岁的美少女";
```

# Java中的保留字

Java之父James Gosling编写的《The Java Programming Language》一书的附录中给出了一个Java的关键字列表。其中包含的一些关键字，例如goto及const，直至今日也并没有真正被Java使用。有人将其称之为Java的保留字。

广义的讲，保留字这个词应有更广泛的含义：在系统类库中使用过的有特殊意义的单词或单词的组合都被视为保留字。

![1.jpg](/images/blog_pic/Java 基础/碎片/1.jpg)

# 标识符(Identifier)

必须以字母|下划线|美元符开头。其他部分可以是字母|下划线|美元符|数字的任意组合。标识符大小写敏感，长度无限。不可为Java的关键字(保留字)。

这里所说的字母，并不仅仅是指针对英语语系的ASCII编码而言。因为实际上Java采用Unicode字符集，因此汉字及其他非英语国家的文字也可出现在标识符中。

示例：

```
String 八云紫 = "我今年17岁";    // 正确
String $ = "想钱想疯了";    // 正确
int 2a = 5;    // 错误，不能以数字开头
int a# = 5;    // 错误，只能包含字母|下划线|美元符|数字，不能包含#这种特殊字符
```

# 字符串连接符

```
System.out.println(4 + 5 + "2");    // 92
```

# 带标签的break和continue

Java中goto是保留字，但却没有相关的跳转的功能。可通过带标签的break和continue实现goto的弱化版功能。不过正如不推荐使用goto那样，自然也不推荐使用带标签的break和continue。

```
public class Test {
    public static void main(String[] args) {
        // 打印[101,150]所有质数
        mark: for (int i = 101; i <= 150; i++) {
            for (int j = 2; j <= Math.sqrt(i); j++) {
                if (i % j == 0) continue mark;
            }
            System.out.println(i);
        }
    }
}
```

# 包关系

com.p1与com.p1.p2并没有从属关系，不过我们通常会从逻辑上认为后者从属于前者。 

# 子类包含的父类中的this指向子类

```
public class Test {

    public static void main(String[] args) {
        new Son().show();
    }
}

class Parent {

    public void m() {
        System.out.println("parent m");
    }

    public void show() {
        this.m();
    }
}

class Son extends Parent {

    @Override
    public void m() {
        System.out.println("son m");
    }
}
```

输出：

```
son m
```

做如下修改：

```
public class Test {

    public static void main(String[] args) {
        new Son().show();
    }
}

class Parent {

    public void m() {
        System.out.println("parent m");
    }

    public void show() {
        this.m();
    }
}

class Son extends Parent {

    @Override
    public void m() {
        System.out.println("son m");
    }

    @Override
    public void show() {
        super.show();
    }
}
```

输出依然为：

```
son m
```

可见，无论如何，子类包含的父类中的this总是指向子类。

# 方法链

以StringBuilder为例：

```
public class Test {

    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder();
        sb.append("八云紫").append("是老婆婆").append(true);
        System.out.println(sb.toString());
    }
}
```

运行后输出：

```
八云紫是老婆婆true
```

这里

```
sb.append("八云紫").append("是老婆婆").append(true);
```

就是一个方法链，其原理为StringBuilder的append方法都实现了类似的结构，我们来随便看一个StringBuilder的append方法的源码：

```
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
```

方法的最后返回了this引用，因此可以像一个链条一样在一行代码中一直点出append方法。