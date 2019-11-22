---
title: Java 基础-重载与重写
date: 2017-10-10 10:36:49
tags: [Java,重载,重写,分派]
categories: Java 基础
---

关于重载和重写的底层实现，可参见[JVM-运行期方法调用](/2017/12/06/JVM-运行期方法调用/)，本文将从解题技巧的角度上分析重载与重写。

# 解题技巧

不妨设计一种通用的场景：

```
G0 o = new G0/G1();
o.m(P p);
```

其中G1为G0的孩子(儿子，孙子，重孙...)。

**总体思路**

- 解重载(Overload)与重写(Override)问题其实就是模拟运行期JVM的方法调用操作：首先，无论如何都会在类加载的连接-解析阶段进行静态多分派(如果需要重载则重载)。随后，如果有必要(实际类型与静态类型不同)，在运行到该调用指令时进行动态单分派(即重写)。

- 静态方法，私有方法，构造方法，被final修饰的方法不可被子类继承，连接方式被称为解析。解析在类加载的连接-解析阶段完成，采用的手段为静态多分派(其间可能会发生重载)。但绝不会有重写(不能被子类继承，自然也不能出现静态类型与实际类型不同的情况)。换句话说，这些方法只会进行解题步骤中的第一步。

- 对于其他能被子类继承的方法而言，除了依然要进行静态多分派之外，在运行到该调用指令时，如果有必要(实际类型与静态类型不同)，还需进行动态单分派(即重写)。因此解题步骤中的两步都要进行。

- 问题的实质其实是确定两个"哪个"：调用"哪个类"的"哪个方法"。

**解题步骤：**

***1.模拟静态多分派***

只考虑方法调用者o的静态类型，本步骤的所有操作都局限在这个类(即例子中的G0)中。在G0中定位出具体的m(P p)，所找寻的方法不仅仅是显式写在G0中的，G0从父类中继承到的也算。若G0中有若干个同名的方法，即需要重载。参数数量及顺序不同导致的重载易于判断，而对于存在继承关系的类型判断，寻找与P p在血缘中最为亲密的那一个。在进行类型判断时，不需要考虑方法参数P的实际类型，只以其静态类型为依据。

实质上，无论是o还是p，该阶段都只需要判断静态类型，因为只要没有实际运行到这一行字节码指令，JVM就无法知道实际类型到底是什么。

一旦该阶段完成，重载便已彻底完成其使命，后续的判断中不再涉及重载，"哪个方法"的问题也已解决一大半。不妨设此阶段决定的方法为G0类中的x，则在本次解题中，G0中对后续操作有意义的方法只有x，其余方法都可以作为干扰项剔除。

本步所模拟的过程是在类加载的连接-解析阶段完成的，其判断依据完全来自于编译期。因此称之为"静态"；本步的判断依据为静态类型及方法参数两个宗量，因此称之为"多分派"。

***2.模拟动态单分派***

如果静态类型与实际类型相同，即不存在重写的可能，则"哪个类"为G0，"哪个方法"为步骤1确定的x。

如果静态类型与实际类型不同，或者具体的说，实际类型是静态类型的子类，则此时存在重写的可能(注意仅仅是可能)。G1继承了源自G0的x(再次强调，只关注x，G0中的其他方法已与本题无关)，若发生了重写，则G1中的x将覆盖从G0中继承的x。换句话说，G1中的x已经是被G1覆盖过之后的了。最终，"哪个类"为实际类型，"哪个方法"为实际类型中的x。

所以简单来说，本步主要回答了以下两点问题：实际类型是否和静态类型一样？若不一样，是否发生了重写？判断重写的规则如下：

- 方法的参数：必须一模一样，包括个数，顺序，类型(有继承关系的子类也不行，必须是一模一样的类)。

- 返回值：一样或为有继承关系的子类。

- 异常检查：对于Checked Exception而言，可以抛出更少的异常，但不能抛出父类中没有定义的异常。对于Unchecked Exception(RuntimeException)及Error而言则没有限制。

- 访问权限：应比父类中的权限更宽松，换句话说，即允许被更多人访问(public > protected > default[即没有修饰] > private)。

本步所模拟的过程是在具体执行到当行调用方法的字节码指令时完成的，因此称之为"动态"；本步的判断依据仅为实际类型这一个宗量(第三次强调，方法x在步骤1已确定，其参数自然也已确定，重写的判断依据之一就是要求参数必须一模一样)，因此称之为"单分派"。

# 例题1

```
public class Test {

    public boolean equals(Test test) {
        System.out.println("Test equals");
        return true;
      }

    public static void main(String[] args) {
        Object o = new Test();
        System.out.println(o.equals(new Test()));
    }
}
```

**输出：**

```
false
```

**分析：**

步骤1，模拟静态多分派:静态类型为Object，调用的方法为equals(Test test)，Object中与之血缘关系最亲近的方法为equals(Object obj)。

步骤2，模拟动态单分派：实际类型为Test，步骤1中确定的方法在Test中没有被重写(方法参数的类型不完全相同)。因此，最终确定被调用的方法为Test类中继承自Object类的equals(Object obj)，或者更准确的说，调用的方法为Object类中的equals(Object obj)。因为该类中的比较是基于对象地址的，因此两个不同的对象的比较结果自然为false。

深入分析一下，做如下修改：

```
public class Test {

    @Override
    public boolean equals(Test test) {
        System.out.println("Test equals");
        return true;
      }

    public static void main(String[] args) {
        Object o = new Test();
        System.out.println(o.equals(new Test()));
    }
}
```

此时无法通过编译，Eclipse下的提示为：

```
The method equals(Test) of type Test must override or implement a supertype method。
```

因此，如果我们试图重写父类方法，那么总是在新方法前面加上@Override是很正确的做法，因为它会在编译阶段就给出检查。

如果，想要重写，那么应做如下修改：

```
public class Test {

    @Override
    public boolean equals(Object obj) {
        System.out.println("Test equals");
        return true;
      }

    public static void main(String[] args) {
        Object o = new Test();
        System.out.println(o.equals(new Test()));
    }
}
```

输出：

```
Test equals
true
```

**分析：**

步骤1，模拟静态多分派:静态类型为Object，调用的方法为equals(Test test)，Object中与之血缘关系最亲近的方法为equals(Object obj)。

步骤2，模拟动态单分派：实际类型为Test，步骤1中确定的方法在Test中被重写，因此，最终确定被调用的方法为Test类中的equals(Object obj)。

# 例题2

```
public class Test {

    public static void main(String[] args) {
        G00 g00 = new G10();
        G10 p = new G10();
        g00.print(p);
    }
}

class G00 {

    public void print(G00 g00) {
        System.out.println("G00 G00");
    }
}

class G10 extends G00 {

    public void print(G00 g00) {
        System.out.println("G10 G00");
    }
}
```

输出：

```
G10 G00
```

**分析：**

步骤1，模拟静态多分派:静态类型为G00，调用的方法为print(G10 p)，G00中与之血缘关系最亲近的方法为print(G00 g00)。

步骤2，模拟动态单分派：实际类型为G10，步骤1中确定的方法在G10中被重写，因此，最终确定被调用的方法为G10类中的print(G00 g00)。

稍加修改：

```
public class Test {

    public static void main(String[] args) {
        G00 g00 = new G10();
        G10 p = new G10();
        g00.print(p);
    }
}

class G00 {

    public void print(G00 g00) {
        System.out.println("G00 G00");
    }

    public void print(G10 g10) {
        System.out.println("G00 G10");
    }
}

class G10 extends G00 {

    public void print(G00 g00) {
        System.out.println("G10 G00");
    }
}
```

输出：

```
G00 G10
```

**分析：**

步骤1，模拟静态多分派:静态类型为G00，调用的方法为print(G10 p)，G00中与之血缘关系最亲近的方法为print(G10 g10))。

步骤2，模拟动态单分派：实际类型为G10，步骤1中确定的方法在G10中没有被重写。因此，最终确定被调用的方法为G10类中继承自G00类的print(G10 g10)，或者更准确的说，调用的方法为G00类中的print(G10 g10)。