---
title: Java 基础-异常
date: 2017-09-29 15:31:49
tags: [Java,异常]
categories: Java 基础
---

首先做一个约定：

- 设计者：可能会抛出Throwable的代码的编写者。

- 使用者：使用代码的人。

所谓异常机制，就是设计者明确自己所写的代码有何风险，应该由自己处理的就自己处理，不该自己处理的抛出给使用者。从而使程序更加健壮。

<!-- more -->

# Throwable

java.lang.Throwable是异常机制的顶层类，常见的java.lang.Error与java.lang.Exception均为其子类。

以Throwable本身为例，Throwable一系通常都有如下两个构造函数：

```
public Throwable()

public Throwable(String message)
```

对于Throwable的实例t而言，直接打印t与打印t.toString()等同。而t.getMessage()则返回生成t时所用的字符串message。

无论是t，t.toString()亦或是t.getMessage()，在打印时都是返回一个字符串。另一个常用的异常打印方法为t.printStackTrace()，该方法的返回值为null，它会在自身的内部将t的堆栈轨迹信息打印至控制台上。

# throw与throws

若程序在执行到某行代码时产生了Throwable，则会自动将其抛出，程序中断。这样虽然简单方便，易于理解，然而却将异常机制的一切都交由JVM处理，问题直至运行期才会暴露出来，不可控性太大。因此Java提供了throw与throws关键字供程序员手动抛出Throwable，从而可以在编写代码时就手动处理这些Throwable。

throw出现在方法体内部的代码逻辑中，可以精确指定程序执行到该行代码时会抛出哪种Throwable。throws则出现在方法定义中。

当程序员在编写代码时遇到了一个在代码中显式抛出的Throwable，都需依序考虑如下两个问题：

1. 是否需要处理这个Throwable？

2. 如果需要，如何处理这个Throwable？

首先需要明确的是，无论是设计者亦或是使用者，无论Throwable抛出的方式是throw还是throws，这两个问题的处理思路都是一致的：

对于第一个问题。Throwable依严重性由大至小被分为Error与Exception，Exception依严重性由大至小又分为受检查异常(Checked Exception)及运行时异常(RuntimeException，为与受检查异常对应，也被称为Unchecked Exception，从这个角度来看，Error都是Unchecked)。所谓受检查，实际指得是受编译器检查：编译器认为这个Throwable非常重要，无论是向更上一层抛出也好，try-catch自行处理也罢，程序员不能无视。而Unchecked则是指编译器不会检查，程序员可自行决定是否无视：Error被设定为Unchecked是因为问题太大程序员已无法处理，而运行时异常被设定为Unchecked是因为问题不大程序员可以不用处理。

所谓不处理就是无视，个人并不推荐这种方式。也就是说虽然从语法的角度上讲可以无需处理运行时异常及Error，但均推荐显式处理它们。

然后进入第二个问题。处理方式有以下两种：

1. 应该由自己处理的使用try-catch在内部将这个Throwable消化，对于更上层的观测者而言，他们是感知不到抛出过这个Throwable的。

2. 不应该由自己处理的通过throws抛出给更上层的观测者。 

这个"应该"取决于具体的代码逻辑。通常来说，以AOP形式存在的切面方法，比如与主业务流程无关的工具型类方法，都会将Throwable throws出去。因为主业务流程最好能够掌控全局：知道工具方法可能产生的Throwable，并制定合理的对策，同时也可以在主流程的日志文件中留下记录。而若工具类方法通过try-catch将Throwable自己消化掉，那么主业务流程将完全不知道发生了什么，它只知道我求这个工具类方法办个事，那边回复办不到(有些时候甚至连这点也确认不了。比如工具类方法返回的不是代表成功或失败的标志，而是一个具体的对象，那么若这个对象为null，是无法判断到底是因为出错了返回null还是没出错就该是null)。同时因为主流程拿不到Throwable，那么错误日志也要由工具类方法打印。这样就会使得能够打印错误日志的类增多，错误信息打印得过于松散，不利于日志分析。更本质的来说，错误信息就不该工具类方法打印，工具类方法就是一个个简单的，处理某个小功能的函数，应该是随时复制粘贴到一个新的工程中直接就能用的，不应与日志系统做这么强的耦合。当然也可以要求工具类方法在其返回值内部包装错误信息，不过这么做显然太过复杂了。

# Error

Error是设计者认为的严重错误：这类错误一旦发生，程序便无法继续下去而应停止。这里的重点在于"认为"两个字，也就是说这是设计者的主观判断，而非客观事实。对于优秀的设计者而言，客观上确实会导致程序无法进行下去的错误，例如Java虚拟机栈溢出等严重的JVM错误，系统崩溃，所需内存空间不足等自然应该被设定为Error。不过还有一些Error是设计者纯从业务角度上设置的，比较典型的就是Java的断言机制：满足断言所需的条件从业务的角度上考虑极其重要，只要断言结果失败，程序便需停止。

纯粹从try-catch的机制上考虑，它能捕获所有的Throwable，自然也可以捕获Error，例如：

```
public class Test {

    /**
     * 启动参数：-ea
     * 该参数用于开启断言
     */
    public static void main(String[] args) {
        int a = 0;
        try {
            assert a > 0 : "a小于等于0";
        } catch (Error e) {
            e.printStackTrace();
        }
        System.out.println("继续");
    }
}
```

输出：

```
java.lang.AssertionError: a小于等于0
	at com.test.Test.main(Test.java:8)
继续
```

不过我并不推荐大家在代码中强行使用try-catch处理Error。关于Error，很多人都搞反了一个因果关系：并非因为是Error而不推荐使用try-catch捕获，而是因为设计者不推荐使用try-catch，所以才将其设定为Error。因此如果在编码中遇到了需要捕获Error的场景，最佳的解决方案应该是联系设计者将其改为Exception：因为此时这个问题已经是可处理的了，应该从本质上修改它的含义，而非由使用者强制捕获。

那么Java异常机制又为什么允许try-catch捕获Error呢？为什么不从语法上干脆将其判为非法呢？这样不是更符合Error的初衷吗？这样不是能迫使更多的程序员遵守良好的规范吗？个人推测原因主要有以下两点：

其一，虽然程序必须因Error而停止，但是允许使用者在停止前做点收尾工作还是很合理的。实际上，Error最为常见的处理方式就是用try-catch捕获，在完成收尾工作后再在catch中将其throw出去。

其二，Throwable是否严重依靠的是设计者的主观判断，而设计者和使用者往往不是一个人，有着不同的身份，对同样一个问题的严重性的判断自然也不同。

我们不妨用一个小例子来说明这个问题。假如程序员A写了工具类方法m，该方法可能会抛出Error。又有程序员B，C，D使用A的m。某天B找到A，说你这个m抛出的那个错误吧，在我这看没什么大不了的，顶多算个Exception，要不你给改成Exception吧。A说那可不行，m是我写的，到底怎么样算严重错误自然应该我来把控，我抛出了Error就说明它确实就是有这么严重。而且不仅是我这么想，你看C，D他们俩也没有意见啊，难道我能为了你而让C，D他俩难办吗？B一想确实是这么个理，便自己用try-catch机制捕获并处理了这个Error。

上例虽然打破了前文说的那个规范，但却不能因此说规范便不重要了。恰恰相反，这正证明了规范的重要之处：若B不遵守规范，他就不会去找A并希望A将Error改为Exception，而是直接就用try-catch将Error处理了。虽然结果上来看是一样的，但是过程却有巨大的不同：上例中B最后捕获并处理Error的前提是建立在他去找了A并进行了细致的沟通，可以说这是一个大家都认可的结果。能做到这一点便已经完成了整个Java异常机制中最重要的目标：让使用者明确他们所欲使用的代码有何风险，并做出合理的应对。正如异常机制的顶层类Throwable的名字所表达的：所谓异常就是设计者将自己的代码可能的问题"抛出"给使用者，让他们心里有点B数。

# Exception

Exception表示设计者认为的，程序可以处理的异常。使用者可以通过捕获异常将灾害控制在尽量小的范围内。

和上文说的Error同理，Exception的这个"可处理"依然是设计者的主观判断。换句话说，围绕Throwable类所构建的整个异常机制都来自于设计者的主观判断，所以使用者其实是默认相信设计者的判断的。如果设计者对问题严重性的判断有误，那么自然就会影响到整个代码的健壮性。

Throwable依严重性由大至小被分为Error与Exception，Exception依严重性由大至小又分为受检查异常(Checked Exception)及运行时异常(RuntimeException)。

Exception有一个名为java.lang.RuntimeException的子类，所有继承自RuntimeException的类都被称为运行时异常，其他的Exception的子类被称为受检查异常。

程序员可以无视被抛出的运行时异常，却必须显式的处理被抛出的受检查异常：使用try-catch自行处理或者使用throws抛给更上一层。

例如以下代码：

```
public class Test {

    public static void main(String[] args) {
        Thread.sleep(-100);
    }
}
```

此时无法通过编译，Eclipse下提示

```
Unhandled exception type InterruptedException。
```

而java.lang.Thread类的sleep方法是这样的：

```
public static native void sleep(long millis) throws InterruptedException;
```

InterruptedException就是设计者显式声明的受检查异常。换句话说，就是设计者在提醒使用者：我的代码可能会出如下的问题！你必须想好合理的应对措施。那么作为使用者，我们可以这样做：

```
public class Test {

    public static void main(String[] args) throws InterruptedException {
        Thread.sleep(-100);
    }
}
```

此时，使用者不处理这个异常，而是将其向更高的层次抛去。在上例中，调用位置main方法已经是顶级层次了，因此这样写就相当于没管这个异常，设计者是否将其抛出结果都一样。不过类似于上文中提到的ABCD四个程序员的例子，即便如此，设计者抛出异常InterruptedException依然是有意义的：因为此时使用者将该异常继续向上抛出是在明确了这个风险后思考的结果，而非真的是在什么都不知道的情况下什么都不做。

当然，我们也可以选择处理这个异常：

```
public class Test {

    public static void main(String[] args) {
        try {
            Thread.sleep(-100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

然后我们运行上述代码，输出：

```
Exception in thread "main" java.lang.IllegalArgumentException: timeout value is negative
	at java.lang.Thread.sleep(Native Method)
	at test.Test.main(Test.java:7)
```

try-catch并未生效，程序还是因异常而中断了。因为此时抛出的异常并非设计者显示抛出的那个InterruptedException，而是IllegalArgumentException：时间当然不可能是负数。这是一个运行时异常。

这种未被使用者显式处理的异常就像是一颗炸弹，只有当程序产生异常时才会因无法处理而挂到，这实在是一件让人很遗憾的事：上线新功能后观察了1个小时运行情况，哦耶一切正常，然后开开心心的下班回家，半夜的时候出现了异常，服务跪掉，然后被领导一个电话拎起来跪着改bug直到天明。

那么，设计者能否将自身代码所有可能产生的问题都显式声明出来呢(写在注释里或在方法定义中throws出去)？这显然是不可能做到的，没有人可以在编码阶段就考虑到所有情况，算无遗策，设计者只能根据他自身的判断，尽可能的将必须处理的问题挑出来。

不过上面那个半夜起床改bug的问题实在是过于让人忧郁了，为此伟大的前辈们总结了这样一个套路：

```
public class Test {

    public static void main(String[] args) {
        try {
            Thread.sleep(-100);
        } catch (InterruptedException e) {
            System.out.println("捕获并处理InterruptedException");
        } catch (IllegalArgumentException e) {
            System.out.println("捕获并处理IllegalArgumentException");
        } catch (Exception e) {
            System.out.println("捕获并处理未知的Exception" + e);
        }
    }
}
```

输出：

```
捕获并处理IllegalArgumentException
```

这其实是利用了Java的继承机制，其思路很明显：先处理设计者显式声明的异常，随后如果使用者有什么不放心的，也可在与设计者沟通或者看设计者提供的文档后写几个自己认为可能会出现的异常。最后用所有异常类的父类Exception兜底，将可能产生的运行时异常一网打尽，并用一个公共的，稳妥的方式处理它。其效果拔群：起码基本是不会在半夜被拎起来了，而是可以在第二天偷摸的通过日志发现异常，然后再偷摸的改掉这个bug。

当然，如果某些异常的处理策略确实就是相同的，也可将它们写到一起：

```
public class Test {

    public static void main(String[] args) {
        try {
            Thread.sleep(-100);
        } catch (InterruptedException | IllegalArgumentException e) {
            System.out.println("捕获并处理" + e);
        } catch (Exception e) {
            System.out.println("捕获并处理未知的Exception" + e);
        }
    }
}
```

输出：

```
捕获并处理java.lang.IllegalArgumentException: timeout value is negative
```

那么又有人说了，用Exception兜底很稳啊，那还费什么劲，只捕获Exception不行吗：

```
public class Test {

    public static void main(String[] args) {
        try {
            Thread.sleep(-100);
        } catch (Exception e) {
            System.out.println("捕获并处理未知的Exception:" + e);
        }
    }
}
```

输出：

```
捕获并处理未知的Exception:java.lang.IllegalArgumentException: timeout value is negative
```

这样做确实可以让代码更简洁，然而问题也很致命：使用者只能用公共的处理模块处理所有异常，不能根据不同的异常设置不同的对策了。因此在这里我还是建议大家不要怕麻烦，除非某些异常确实就是要用通用的模块处理，否则还是单独写处理逻辑更利于维护。

# 异常机制使用技巧

首先我们来看如下这段代码：

```
public class Test {

    public static void main(String[] args) {
        int a = 0;
        int b = 100;
        if (a == 0) System.out.println(b / (a + 1));
        else System.out.println(b / a);
    }
}
```

当然也可以这样写：

```
public class Test {

    public static void main(String[] args) {
        int a = 0;
        int b = 100;
        try {
            System.out.println(b / a);
        } catch (ArithmeticException e) {
            System.out.println(b / (a + 1));
        }
    }
}
```

这两种写法均能实现相同的功能，看似没有什么区别，但是事实上，写代码不仅仅要看当前的业务逻辑，更要遵守语法本身的基本规范。第二段代码最本质的问题根本就不是什么异常机制占用资源高，降低程序性能之类的具体原因，因为如果问题是这些的话，那么是不是说如果程序不大，机器硬件资源很充足，第二段代码就没问题了？当然不是，第二段代码最本质的问题在于它违反了Java异常机制的设计初衷，这两段代码中的除零根本就不是异常，而是一个正常的业务逻辑，换句话说，尽量不要用异常机制去处理正常逻辑的代码，即不要把异常机制当成正常逻辑下的条件分支去用。这里之所以用了尽量，是因为写代码也要有灵性，某些特殊情况下取个巧，将异常机制融入正常逻辑会更好。不过上例显然不属于这种情况。

---

正如上文所说的不要用异常机制处理正常的业务逻辑那样，Throwable终究是一种无可避免的情况下产生的不正常的东西，所以能用正常业务代码处理的就不要显式声明为Throwable。例如有人不进行边界判断，而是直接用异常机制去控制非法的方法入参，这是不可取的。比如如下代码：

```
public class Test { 

    public static int m(int a, int b) {
        return a / b;
    }

    public static void main(String[] args) {
        int c = Test.m(1, 0);
    }
}
```

输出：

```
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at com.test.Test.m(Test.java:6)
	at com.test.Test.main(Test.java:10)
```

显然这是最糟的写法，调用者对于m可能会抛出ArithmeticException这一运行时异常完全没有准备。因此自然而言想要这么写：

```
public class Test { 

    public static int m(int a, int b) {
        if (b == 0) {
            System.out.println("除数不能为0，m执行失败");
            return 0;
        }
        return a / b;
    }

    public static void main(String[] args) {
        int c = Test.m(1, 0);
    }
}
```

输出：

```
除数不能为0，m执行失败
```

我想，大家在看到方法m里的那句输出时心里可能就会隐隐觉得不妥了，这代码似乎依然不怎么样。没错，这种处理方式对于调用者而言异常是透明的。那么怎么写比较好呢？笔者比较认可如下的写法：

```
public class Test { 

    /**
     * @throws ArithmeticException
     */
    public static int m(int a, int b) {
        if (b == 0) throw new ArithmeticException();
        return a / b;
    }

    public static void main(String[] args) {
        int c = Test.m(1, 0);
    }
}
```

输出：

```
Exception in thread "main" java.lang.ArithmeticException
	at aaa.Test.m(Test.java:6)
	at aaa.Test.main(Test.java:11)
```

需要注意的是，上述代码是先进行了边界判断，再依具体的代码逻辑选择抛出ArithmeticException这一具体的异常。和直接使用异常机制代替边界判断是不同的。

---

正如我所信奉的那句仿佛是出自于圣经一般的格言：Thou Shalt Comment(汝应注释)所说，设计者一定要为异常写好注释。

---

catch代码块的内容不要为空。最不济也要打一行日志，记录下错误。

# 自定义异常

优先使用Java API提供的标准Throwable，尽量不要自造Throwable。因为设计者只要自造了Throwable，使用者就必须引入这个Throwable，平添复杂性。实际上，根据我自身的经验，Java API提供的标准Throwable基本上已经能满足所有业务需求了。如果实在是需要自造Throwable，建议也只是自造Exception。

在真正开始自造Exception之前，我们不妨以IOException为例，看看Java API中的异常是怎么写的。因为代码都很短，我们可以从更上层的Exception开始贴起(Throwable就有些长了)

Exception:

```
package java.lang;

public class Exception extends Throwable {
    static final long serialVersionUID = -3387516993124229948L;

    public Exception() {
        super();
    }

    public Exception(String message) {
        super(message);
    }

    public Exception(String message, Throwable cause) {
        super(message, cause);
    }

    public Exception(Throwable cause) {
        super(cause);
    }

    protected Exception(String message, Throwable cause,
                        boolean enableSuppression,
                        boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

IOException:

```
package java.io;

public
class IOException extends Exception {
    static final long serialVersionUID = 7818375828146090155L;

    public IOException() {
        super();
    }

    public IOException(String message) {
        super(message);
    }

    public IOException(String message, Throwable cause) {
        super(message, cause);
    }

    public IOException(Throwable cause) {
        super(cause);
    }
}
```

实在是有趣！

一共没几行代码不说，还没有任何独有的逻辑，全部都是super。看来Java API的思路就是在Throwable中设计一个套路，然后它的小弟们都沿着这个套路走。仔细想想确实也够用了，我们在查看异常信息时需要的无非就是异常类型及描述字符串。

仿造IOException，我们很容易的就可以写出一个自定义异常：

```
import java.io.IOException;

public class Test {

    public static void main(String[] args) throws MyIOException {
        throw new MyIOException("自造的IOException");
    }
}

class MyIOException extends IOException {

    private static final long serialVersionUID = 1L;

    public MyIOException() {
        super();
    }

    public MyIOException(String message) {
        super(message);
    }

    public MyIOException(String message, Throwable cause) {
        super(message, cause);
    }

    public MyIOException(Throwable cause) {
        super(cause);
    }
}
```

输出：

```
Exception in thread "main" com.test.MyIOException: 自造的IOException
	at com.test.Test.main(Test.java:8)
```

默认调用的是toString()方法，它位于Throwable中：

```
public String toString() {
    String s = getClass().getName();
    String message = getLocalizedMessage();
    return (message != null) ? (s + ": " + message) : s;
}
```

我们当然也可以重写它：

```
import java.io.IOException;

public class Test {

    public static void main(String[] args) throws MyIOException {
        throw new MyIOException("描述信息");
    }
}

class MyIOException extends IOException {

    private static final long serialVersionUID = 1L;

    public MyIOException() {
        super();
    }

    public MyIOException(String message) {
        super(message);
    }

    public MyIOException(String message, Throwable cause) {
        super(message, cause);
    }

    public MyIOException(Throwable cause) {
        super(cause);
    }

    @Override
    public String toString() {
        String s = "一种特殊的IOException";
        String message = super.getLocalizedMessage();
        return (null != message) ? (s + ": " + message) : s;
    }
}
```

输出：

```
Exception in thread "main" 一种特殊的IOException: 描述信息
	at com.test.Test.main(Test.java:8)
```

# try-catch-finally中的return

```
public class Test {

    private static String test() {
        String result = null;
        try {
            result = "try";
            System.out.println(result);
            return result;
        } catch (Exception e) {
            result = "catch";
            System.out.println(result);
            return result;
        } finally {
            result = "finally";
            System.out.println(result);
        }
    }

    public static void main(String[] args) {
        System.out.println("method result = " + Test.test());
    }
}
```

输出：

```
try
finally
method result = try
```

上例的执行顺序为：

1. return前将待return的值保存。

2. 将待return的值复制一份给finally。

3. 执行finally。类似于[Java 基础-方法参数的值传递及引用传递](/2017/10/10/Java 基础-方法参数的值传递及引用传递/)。finally中所有的操作都是基于这份复制后的新值，与原有待返回的值无关。

4. 执行返回。

那么上述执行顺序的底层实现是什么样的呢？我们再来看如下代码：

```
public class Test {

    public int inc() {
        int x;
        try {
            x = 1;
            return x;
        } catch (Exception e) {
            x = 2;
            return x;
        } finally {
            x = 3;
        }
    }
}
```

为从字节码层面分析该问题，我们先用javap生成该类class文件的反编译文件：

```
Classfile /D:/Test.class
  Last modified 2017-11-17; size 397 bytes
  MD5 checksum c86c2f92295c441780b75728395e2c1f
  Compiled from "Test.java"
public class Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#16         //  java/lang/Object."<init>":()V
   #2 = Class              #17            //  java/lang/Exception
   #3 = Class              #18            //  Test
   #4 = Class              #19            //  java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Utf8               Code
   #8 = Utf8               LineNumberTable
   #9 = Utf8               inc
  #10 = Utf8               ()I
  #11 = Utf8               StackMapTable
  #12 = Class              #17            //  java/lang/Exception
  #13 = Class              #20            //  java/lang/Throwable
  #14 = Utf8               SourceFile
  #15 = Utf8               Test.java
  #16 = NameAndType        #5:#6          //  "<init>":()V
  #17 = Utf8               java/lang/Exception
  #18 = Utf8               Test
  #19 = Utf8               java/lang/Object
  #20 = Utf8               java/lang/Throwable
{
  public Test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 1: 0

  public int inc();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=5, args_size=1
         0: iconst_1  // 将int型1压入操作数栈
	              // 操作数栈(自底向上)：1
		      // 局部变量表：this

         1: istore_1  // 将栈顶int型数值弹出并存入局部变量表1号索引处
	              // 操作数栈(自底向上)：无
		      // 局部变量表：this,1

         2: iload_1   // 将局部变量表1号索引位置的int型数值压入操作数栈
	              // 操作数栈(自底向上)：1
		      // 局部变量表：this,1

         3: istore_2  // 将栈顶int型数值弹出并存入局部变量表2号索引处
	              // 操作数栈(自底向上)：无
		      // 局部变量表：this,1,1

         4: iconst_3  // 将int型3压入操作数栈
	              // 操作数栈(自底向上)：3
		      // 局部变量表：this,1,1

         5: istore_1  // 将栈顶int型数值弹出并存入局部变量表1号索引处
	              // 操作数栈(自底向上)：无
		      // 局部变量表：this,3,1
	 
         6: iload_2   // 将局部变量表2号索引位置的int型数值压入操作数栈
	              // 操作数栈(自底向上)：1
		      // 局部变量表：this,3,1
	 
         7: ireturn   // 返回操作数栈栈顶的int类型数值
	 
         8: astore_2  // [0,3]产生异常跳入本行，跳入时默认会将异常对象压入操作数栈
	              // 假设压入异常对象之前的操作数栈及局部变量表保持指令2结束时的样子。
		      // 本指令为：将栈顶引用型值弹出并存入局部变量表2号索引处
		      // 操作数栈(自底向上)：1
		      // 局部变量表：this,1,e
	 
         9: iconst_2  // 将int型2压入操作数栈
		      // 操作数栈(自底向上)：1,2
		      // 局部变量表：this,1,e  

        10: istore_1  // 将栈顶int型数值弹出并存入局部变量表1号索引处
		      // 操作数栈(自底向上)：1
		      // 局部变量表：this,2,e    

        11: iload_1   // 将局部变量表1号索引位置的int型数值压入操作数栈
		      // 操作数栈(自底向上)：1,2
		      // 局部变量表：this,2,e     

        12: istore_3  // 将栈顶int型数值弹出并存入局部变量表3号索引处
		      // 操作数栈(自底向上)：1
		      // 局部变量表：this,2,e,2   

        13: iconst_3  // 将int型3压入操作数栈
	              // 操作数栈(自底向上)：1,3
		      // 局部变量表：this,2,e,2

        14: istore_1  // 将栈顶int型数值弹出并存入局部变量表1号索引处
	              // 操作数栈(自底向上)：1
		      // 局部变量表：this,3,e,2  

        15: iload_3   // 将局部变量表3号索引位置的int型数值压入操作数栈
	              // 操作数栈(自底向上)：1,2
		      // 局部变量表：this,3,e,2    

        16: ireturn   // 返回操作数栈栈顶的int类型数值

        17: astore        4  // [0,3]或[8,12]产生异常跳入本行，跳入时默认会将异常对象压入操作数栈
                             // 假设压入异常对象之前的操作数栈及局部变量表保持指令11结束时的样子。
			     // 本指令为：将栈顶引用型值弹出并存入局部变量表4号索引处
			     // 操作数栈(自底向上)：1,2
		             // 局部变量表：this,2,e,空,e2
			     // 其中e为try中抛出的异常，e2为catch中抛出的异常

        19: iconst_3  // 将int型3压入操作数栈
		      // 操作数栈(自底向上)：1,2,3
		      // 局部变量表：this,2,e,空,e2

        20: istore_1  // 将栈顶int型数值弹出并存入局部变量表1号索引处
		      // 操作数栈(自底向上)：1,2
		      // 局部变量表：this,3,e,空,e2

        21: aload         4  // 将局部变量表4号索引位置的引用型值压入操作数栈
			     // 操作数栈(自底向上)：1,2,e2
		             // 局部变量表：this,3,e,空,e2

        23: athrow  // 将操作数栈栈顶的异常抛出
      Exception table:
         from    to  target type
             0     4     8   Class java/lang/Exception
             0     4    17   any
             8    13    17   any
            17    19    17   any
      LineNumberTable:
        line 6: 0
        line 7: 2
        line 12: 4
        line 8: 8
        line 9: 9
        line 10: 11
        line 12: 13
      StackMapTable: number_of_entries = 2
           frame_type = 72 /* same_locals_1_stack_item */
          stack = [ class java/lang/Exception ]
           frame_type = 72 /* same_locals_1_stack_item */
          stack = [ class java/lang/Throwable ]

}

```

程序可能的执行流程及返回结果有如下4种：

try中未产生异常，则流程为try-finally。返回1。即：

- [0,3]:try

- [4,5]:finally

- [6,7]:try返回1。

try中产生Exception及其子类的异常则进入catch，若catch未产生异常，则进入finally，返回2。

- [0,x]:try,x取值范围为[0,3]

- [8,12]:catch

- [13,14]:finally

- [15,16]:catch返回2。

try中产生Exception及其子类的异常则进入catch，若catch产生异常，则进入finally，而后抛出该异常，无返回值。

- [0,x]:try,x取值范围为[0,3]

- [8,y]:catch，y取值范围为[8,12]

- [17]:记录异常

- [19,20]:finally

- 21,23:抛出异常，无返回值

try中产生不属于Exception及其子类的异常，则进入finally，而后抛出该异常，无返回值。

- [0,x]:try,x取值范围为[0,3]

- [17]:记录异常

- [19,20]:finally

- 21,23:抛出异常，无返回值

在JDK1.4.2之前JVM采用jsr及ret指令来实现finally。JDK1.4.2中虽然jsr及ret仍有效，但javac编译器已经不再生成了这两个指令了，它会在finally可能出现的所有路径上冗余添加finally代码。而到了JDk1.7，jsr及ret已被完全禁用，若class文件中出现了这两个指令，JVM会在类加载-连接-验证-字节码校验阶段抛出异常。

实例代码的执行流程分析也证明了这一点。[4,5]，[13,14]，[19,20]就是冗余存储的finally逻辑。

这样我们便从字节码的层面上验证了最开始描述的执行顺序，同时还知道了不仅try到finally对于return值会采取值传递的方式，catch到finally时也一样。

因为finally会被无条件执行，所以最好不要在finally中使用return语句：虽然语法上没什么问题，却会造成逻辑混乱，不利于代码的维护。

---

最后我们可以再一起来做一个小练习：

```
public class Test {

    public static void main(String[] args) {
        try {
            throw new ESon();
        } catch (ESon e) {
            System.out.println("ESon");
        } catch (EFather e) {
            System.out.println("EFather");
        }
    }
}

class EFather extends Exception {

    private static final long serialVersionUID = 6536621933798974349L;
}

class ESon extends EFather {

    private static final long serialVersionUID = 6091571391764298719L;
}
```

此时Eclipse下程序会报警告：

```
Unreachable catch block for EFather. Only more specific exceptions are thrown and they are handled by previous catch block(s).
```

输出：

```
ESon
```

若将catch的顺序交换，即：

```
public class Test {

    public static void main(String[] args) {
        try {
            throw new ESon();
        } catch (EFather e) {
            System.out.println("EFather");
        } catch (ESon e) {
            System.out.println("ESon");
        }
    }
}

class EFather extends Exception {

    private static final long serialVersionUID = 6536621933798974349L;
}

class ESon extends EFather {

    private static final long serialVersionUID = 6091571391764298719L;
}
```

程序将无法通过编译，Eclipse下报：

```
Unreachable catch block for ESon. It is already handled by the catch block for EFather。
```

若按如下修改：

```
public class Test {
 
    public static void main(String[] args) throws Exception {
        try {
            try {
                throw new ESon();
            } catch (EFather f) {
                System.out.println("Caught Father");
                throw f;
            }
        } catch (ESon s) {
            System.out.println("Caught Son");
            return;
        } finally {
            System.out.println("Hello World!");
        }
    }
}

class EFather extends Exception {

    private static final long serialVersionUID = 6536621933798974349L;
}

class ESon extends EFather {

    private static final long serialVersionUID = 6091571391764298719L;
}
```

输出：

```
Caught Father
Caught Son
Hello World!
```

# try-catch-finally常用情景：文件读写

按照上文讨论的结果，Throwable是由代码的设计者抛出的。其中一个很常见的应用场景就是文件的读写。

之所以这么说，是因为文件读写的程序出了错，很多时候锅往往不会在代码本身：比如要操作的文件没有读写权限，或者干脆就没有。对于设定好要读写文件的代码而言，这确实是异常情况，而且无论代码写得多么健壮都无法避免。

下面我们来看一个简单的小例子：

```
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class Test {

    public static void main(String[] args) {
        FileReader fileReader = null;
        try {
            fileReader = new FileReader("d:/a.txt");
        } catch (FileNotFoundException e) {
            System.out.println("异常点1");
            e.printStackTrace();
            return;
        }
        try {
            // 方法返回int，强转为char
            System.out.println((char)fileReader.read());
        } catch (IOException e) {
            System.out.println("异常点2");
            e.printStackTrace();
        }
        try {
            if(null != fileReader) fileReader.close();
        } catch (IOException e) {
            System.out.println("异常点3");
            e.printStackTrace();
        }
    }
}
```

然后我们创建d:/a.txt，并在其中写入内容"博丽灵梦"。然后运行程序，输出为:

```
博
```

这是符合预期的正常情况，没毛病。不过，如果我们删掉d盘下的a.txt，再运行程序，将输出：

```
异常点1
java.io.FileNotFoundException: d:\a.txt (系统找不到指定的文件。)
	at java.io.FileInputStream.open(Native Method)
	at java.io.FileInputStream.<init>(FileInputStream.java:146)
	at java.io.FileInputStream.<init>(FileInputStream.java:101)
	at java.io.FileReader.<init>(FileReader.java:58)
	at com.test.Test.main(Test.java:12)
```

上述代码已可以健壮的实现所需功能了，不过还有可以改进的地方：文件的关闭被独立出去了，如果分支处理不当可能会有漏关的风险。而实际上它与创建应是一一对应的关系。我们可以使用finally的特性来确保这一点：

```
import java.io.FileReader;
import java.io.IOException;

public class Test {

    public static void main(String[] args) {
        FileReader fileReader = null;
        try {
            fileReader = new FileReader("d:/a.txt");
           // 方法返回int，强转为char
            System.out.println((char)fileReader.read());
        } catch (IOException e) {
            // FileNotFoundException是IOException的子类，因此合并了
            System.out.println("异常点1");
            e.printStackTrace();
            return;
        } finally {
            try {
                if(null != fileReader) fileReader.close();
            } catch (IOException e) {
                System.out.println("异常点2");
                e.printStackTrace();
            }
        }
    }
}
```

看起来好多了。不过我们仍然可以让它更为简洁。因为对于文件操作这一类的资源读取而言，关闭读入源是如此的常见，因此语言的API中往往会提供对应的快捷方式。比如Python中就有对应的try-with，可以在资源使用完毕后(代码执行出try的范围)由系统自动关闭资源，而无需程序员显式设置。在JDK1.7中，Java终于也提供了类似的功能：

```
import java.io.FileReader;
import java.io.IOException;

public class Test {

    public static void main(String[] args) {
        try (FileReader fileReader = new FileReader("d:/a.txt");) {
           // 方法返回int，强转为char
            System.out.println((char)fileReader.read());
        } catch (IOException e) {
            // FileNotFoundException是IOException的子类，因此合并了
            System.out.println("异常点1");
            e.printStackTrace();
            return;
        }
    }
}
```