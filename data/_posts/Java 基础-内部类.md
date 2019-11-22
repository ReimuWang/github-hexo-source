---
title: Java 基础-内部类
date: 2017-09-28 14:57:49
tags: [Java,内部类]
categories: Java 基础
---

内部类可分为如下3类：成员内部类，局部内部类，匿名内部类。而成员内部类又可分为实例成员内部类及静态成员内部类。

<!-- more -->

# 成员内部类

**实例成员内部类**

```
public class Out {

    private class In {
    }

    public static void main(String[] args) {
        new Out().new In();
    }
}
```

实例成员内部类中是无法定义类变量的：

```
public class Out {

    private class In {

        private static int V = 1;
    }
}
```

这段代码无法通过编译，提示

```
The field V cannot be declared static in a non-static inner type, unless initialized with a constant 
expression。
```

**静态成员内部类**

```
public class Out {

    private static class In {
    }

    public static void main(String[] args) {
        new Out.In();
    }
}
```

**成员内部类的访问权限**

成员内部类，顾名思义，首先是个成员(即实例变量或类变量)，其次才是类。从更大的视角上看，成员内部类就是其外部类的一个成员。因此二者其实是同一个类。进而二者均有权限访问对方内部被private修饰的字段或方法。

下面以外部类访问静态成员内部类中的私有变量为例：

```
public class Out {

    private static class In {

        private String v0 = "实例";

        private static String V_1 = "静态";
    }

    public static void main(String[] args) {
        System.out.println(Out.In.V_1);
        System.out.println(new Out.In().v0);
    }
}
```

输出：

```
静态
实例
```

外部类(其实也就是普通的Java类)的修饰符只能为public或无修饰符。而成员内部类本质上是成员，因此同其他变量一样，可被private,protected,无修饰符,public修饰，并且访问权限也与一般变量一般无二。

**内部类与外部类之间同名字段问题**

在叙述具体的解决策略之前，我想先表达一下个人看法。如果我是Java语法的设计者，那么我可能会禁止内部类与外部类之间存在同名字段。因为这与因继承导致的重复不同：继承牵扯到复数个类，同时父类与子类之间又有明确的ISA关系，因此允许字段重名是合理的。而内部类与外部类之间则是HASA的关系，二者并没有什么血缘关系，从逻辑上无需重名。同时内部类与外部类共存于一个Java源文件中，也基本不会造成因需要兼顾的代码量太大而漏掉字段的可能，毕竟需要检查的代码范围也就是一个Java源文件那么大，综上，我认为允许内部类与外部类的字段重名稍稍有些灵活过了(通常Java在和C++做比较时，没那么灵活往往是Java的优势，可以让程序员节约很多学习成本)。同时，我也建议大家在编码时，即便没有编译器的强制规范，也不要编写出重名的字段。

若发生了重名，如果什么都不处理的话，则会默认调用当前代码所在类的字段：

```
public class Out {

    private String v0 = "外部实例";

    private static String V_1 = "外部静态";

    private static class In {

        private String v0 = "内部实例";

        private static String V_1 = "内部静态";

        private void m0() {
            System.out.println(v0);
        }

        private static void m1() {
            System.out.println(V_1);
        }
    }

    public static void main(String[] args) {
        System.out.println("外部类中调用情况");
        System.out.println(new Out().v0);
        System.out.println(V_1);
        System.out.println("==========================");
        System.out.println("内部类中调用情况");
        new Out.In().m0();
        Out.In.m1();
    }
}
```

输出：

```
外部类中调用情况
外部实例
外部静态
==========================
内部类中调用情况
内部实例
内部静态
```

如果我就是想在外部类中调内部类的字段，内部类中调外部类的字段呢？

```
public class Out {

    private String v0 = "外部实例";

    private static String V_1 = "外部静态";

    private static class In {

        private String v0 = "内部实例";

        private static String V_1 = "内部静态";

        private static void m1() {
            System.out.println(Out.V_1);
        }
    }

    public static void main(String[] args) {
        System.out.println("外部类中调用情况");
        System.out.println(new Out.In().v0);
        System.out.println(Out.In.V_1);
        System.out.println("==========================");
        System.out.println("内部类中调用情况");
        Out.In.m1();
    }
}
```

此时该成员内部类是无法访问到其外部类的实例变量，究其原因，还是因为该成员内部类是静态成员内部类。因此我们略作修改：

```
public class Out {

    private String v0 = "外部实例";

    private class In {

        private String v0 = "内部实例";

        private void m0() {
            System.out.println(Out.this.v0);
        }
    }

    public static void main(String[] args) {
        Out out = new Out();
        Out.In in = out.new In(); 
        in.m0();
    }
}
```

输出：

```
外部实例
```

# 局部内部类

局部内部类是定义在一个方法或者一个作用域里面的类。如果说从宏观上来看成员内部类的本质是字段，那么局部内部类的本质就是局部变量。

因为本质是局部变量，那么对于局部内部类而言，自然不存在访问修饰符的概念。不过，局部内部类中的私有数据与其外部类的私有数据之间依然是相互可见的。

局部内部类中不能定义类变量，即便其所处的外部类方法是静态的也不行：

```
public class Out {

    private static void m() {
        class In {
            private static String v_1_I = "内部静态";
        }
    }
}
```

无法通过编译，提示

```
The field v_1_I cannot be declared static in a non-static inner type, unless initialized with a constant。
```

测试访问权限的代码如下：

```
public class Out {

    private String vo = "外部实例";

    private static String V = "外部静态";

    private void m() {
        class In {
            private String vi = "内部实例";

            public void mi(Out out) {
                System.out.println(out.vo);
                System.out.println(Out.V);
            }
        }
        In in = new In();
        System.out.println("外部类中调用情况");
        System.out.println(in.vi);
        System.out.println("==========================");
        System.out.println("内部类中调用情况");
        in.mi(this);
    }

    public static void main(String[] args) {
        new Out().m();
    }
}
```

输出：

```
外部类中调用情况
内部实例
==========================
内部类中调用情况
外部实例
外部静态
```

# 匿名内部类

匿名内部类(Anonymous Inner Class)可以继承其他类或实现其他接口，在Swing编程和Android开发中常用此方式来实现事件监听和回调。

下面是一个小例子：

```
public abstract class Test {

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
            }
        }).start();
    }
}
```

匿名内部类是唯一一种没有构造器的类。正因为其没有构造器，所以匿名内部类的使用范围非常有限，大部分匿名内部类用于接口回调。上例中的匿名内部类在编译的时候由系统自动起名为Test$1.class。一般来说，匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。

# final修饰符问题

局部内部类或匿名内部类中只能引用其所在方法中被final修饰的局部变量：

```
public class Test {

    public void test(final int b) {
        final int a = 10;
        new Thread(){
            public void run() {
                System.out.println(a);
                System.out.println(b);
            };
        }.start();
    }
}
```

上述代码，如果去掉b或a的final修饰符，将无法通过编译，并提示

```
Cannot refer to the non-final local variable a defined in an enclosing scope。
```

究其原因，匿名内部类或局部内部类在使用外部局部变量时，依靠的是将其传入自身的实例构造函数方法&lt;init&gt;。我们知道Java在进行方法传值时使用的是值传递(详见[Java 基础-方法参数的值传递及引用传递](/2017/10/10/Java 基础-方法参数的值传递及引用传递/))，这样可以保证即便是在本例这样的并发环境下，test方法可能已执行完成，局部变量a或b已被销毁，匿名内部类所起的线程中的a或b依然可用：因为这已然是个全新的复制值了。

但是这样便引入了新的问题：即若a或b在复制了一份传入匿名内部类或局部内部类后发生了改变怎么办？为了保证线程安全，才强制规定局部内部类或匿名内部类中只能引用其所在方法中被final修饰的局部变量。