---
title: Java基础-基本数据类型
date: 2017-09-27 15:03:49
tags: [Java,基本数据类型]
categories: Java 基础
---

<!-- more -->

# 4类8种基本数据类型

除此之外，特别的，Java中存在一种基本数据类型void，它所对应的包装类为java.lang.Void。不过我们无法直接对它进行操作。

**四种整数类型**

- byte：8位有符号 [-2^7 , 2^7-1] --> [-128 , 127]
- short：16位有符号 [-2^15 , 2^15-1] --> [-32768 , 32767]
- int：32位有符号 [-2^31 , 2^31-1] --> [-2,147,483,648 , 2,147,485,647]。默认整数型
- long：64位有符号 [-2^63 , 2^63-1] --> [-9,223,372,036,854,775,808 , 9,223,372,036,854,775,807]。可用L或l显式声明

若需要表示long范围之外的数字，可使用java.math.BigInteger，其可表示任意长度的整数。

默认采用十进制计数。0开头为八进制。0x或0X开头为16进制。进制间转换示例：

```
public class Test {

    public static void main(String[] args) {
        int a = 17;
        System.out.println("2进制=" + Integer.toBinaryString(a));    // 转为2进制字符串
        System.out.println("8进制=" + Integer.toOctalString(a));    // 转为8进制字符串
        System.out.println("16进制=" + Integer.toHexString(a));    // 转为16进制字符串
    }
}
```

输出:

```
2进制=10001
8进制=21
16进制=11
```

JDK1.7中新增0b/0B开头表示二进制数：

```
public class Test {

    public static void main(String[] args) {
        System.out.println(0b10);
    }
}
```

输出：

```
2
```

JDK1.7中新增数字分割符，可用下划线按照自己的习惯随意分割一个数：

```
int a = 111_222_333;
double b = 123_456.123_45;
```

**两种浮点数类型**

- float：32位有符号单精度。[1.4E-45 , 3.4028235E38]。又被称为单精度类型，尾数可以精确到7位有效数字。可用F或f显式声明
- double：64位有符号双精度。[4.9E-324 , 1.7976931348623157E308]。又被称为双精度类型，尾数可以精确到14位有效数字。默认浮点型。可用D或d显式声明

浮点型存在舍入误差(例如，无法精确表示0.1，0.01这种10的负数次方幂)，因此应尽量避免用于比较，若实在需比较浮点数，可使用java.math.BigDecimal，其可精确表示任意精度的浮点数。

```
public class Test {

    public static void main(String[] args) {
        double d = 1.0 / 10;
        System.out.println(d == 0.1);    // true
	System.out.println(d == 0.1F);    // false
	System.out.println(0.1 == 0.1F);    // false
    }
}
```

可用科学计数法表示浮点型：

```
public class Test {

    public static void main(String[] args) {
        System.out.println(3.14e-2);
    }
}
```

输出：

```
0.0314
```

**一种字符类型(char)**

16位无符号[0 , 2^16-1] --> [0, 65535]。可以当整数来用，它的每一个字符都对应一个数字。换言之也可将[0, 65535]的整数转换为char。

```
public class Test {

    public static void main(String[] args) {
        System.out.println((int)'a');
        System.out.println('a' + 1);    // 'a'直接作为一个整数参与四则运算。换句话说，char可自动向上转换为int
        System.out.println((char)97);
    }
}
```

输出：

```
97
98
a
```

char本质上就是个二进制的数，显示的时候处理为字符。char在参与四则运算时会被转换为int型。

char与Java所使用的字符集Unicode同为两字节表示(不考虑扩展集)，一个char对应Unicode的一个字符。

'A'表示字符，是char类型。"A"表示字符串，是String类型，该字符串由一个字符组成。

String类，其实是字符序列(char sequence)。

```
System.out.println('');    // 无法通过编译
```

\u表示Unicode编码值，以16进制表示：

```
System.out.println('\u0061');    // 输出a
```

![0.jpg](/images/blog_pic/Java 基础/基本数据类型/0.jpg)

```
public class Test {

    public static void main(String[] args) {
        // char可用于存储换行等转义字符
        System.out.println('\n');
        System.out.println('\'');
        System.out.println('\\');
    }
}
```

输出：

```


'
\
```

**一种布尔类型(boolean)**

关于boolean的大小，有以下几种说法：

- 1个bit(1/8个字节)：boolean只有true和false两种逻辑值，在编译后会使用1和0来表示，这两个数在内存中按位算，仅需1bit即可存储，位是计算机最小的存储单位。
- 1个字节：虽然编译后1和0只需占用1位空间，但计算机处理数据的最小单位是1个字节，1个字节等于8位，实际存储的空间是：用1个字节的最低位存储，其他7位用0填补，如果值是true的话则存储的二进制为：0000 0001，如果是false的话则存储的二进制为：0000 0000。
- 4个字节：JVM规范中规定：虽然定义了boolean这种数据类型，但是只对它提供了非常有限的支持。在JVM中没有任何供boolean值专用的字节码指令，Java语言表达式所操作的boolean值，在编译之后都使用JVM中的int数据类型来代替，而boolean数组将会被编码成JVM的byte数组，每个元素占8位。JVM使用int而非更小的整型代指boolean的原因为：对于当下32位的CPU而言，一次处理的数据为32位(这里的32不是指操作系统的32位与64位，而是指CPU硬件层面)，因此32位最为合理高效，即便使用了更小的位数最终也会被填充为32位。

# 自动类型转换

![1.jpg](/images/blog_pic/Java 基础/基本数据类型/1.jpg)

上图中黑色的实线表示不会发生精度丢失的自动转换，红色表示可能会发生精度丢失的自动转换(是否精度丢失主要是看转换双方的相对大小)。

**浮点型间的类型转换**

```
float f = 3.4;
```

无法通过编译。Eclipse提示：Type mismatch: cannot convert from double to float。

3.4默认是双精度数，将双精度型(double)赋值给浮点型(float)属于下转型(down-casting)，也可称为窄化。该操作会造成精度损失，因此无法通过编译。

解决办法有2：

```
// 强制类型转换
float f = (float)3.4;

// 显式声明类型
float f = 3.4F;
```

**整型间的类型转换**

```
short s = 1;    // 正确。除非涉及四则运算符，否则可自动关联关键字类型。这与float不同。
```

```
int i = 1;
short s = i;    // 错误，无法通过编译
```

```
// 错误
// 1是int型，s + 1的运算结果为int型，需要强制类型转换才能赋值给short型。
s = s + 1;
```

```
// 正确
// 本句等价于s = (short)(s + 1);其中有隐含的强制类型转换。
// s++;同理也是正确的。
s += 1;
```

```
short s1 = 1;
short s2 = 2;
// 无法通过编译
// Type mismatch: cannot convert from int to short
short s3 = s1 + s2;
```

同理，上例中的short换成byte后相关结果依然成立。

# 基本数据类型与其包装类型

Java并非纯粹的面向对象语言，因此类似于"Java中一切都是对象，对象就是一切"这样的言论更像是宣传口号，实际上是有些绝对了。

比如，Java中的基本数据类型就不是面向对象的(这样的好处是在处理一些涉及基本数据类型的操作时效率更高)，但是有时(比如基本数据类型作为集合类的元素)，我们又希望能够像操作对象那样操作它们。因此，Java为每一个基本数据类型都提供了对应的包装类型(wrapper class)，从JDK1.5开始又引入了自动装箱/拆箱机制，使得二者可以相互转换。

这些包装类型都位于java.lang包中：

- boolean - Boolean
- char - Character
- byte - Byte
- short - Short
- int - Integer
- long - Long
- float - Float
- double - Double

# 自动装箱/拆箱机制

```
Integer a = new Integer(3);
Integer b = 3;    // 将3自动装箱成Integer类型
int c = 3;
System.out.println(a == b);    // false 两个引用没有引用同一对象
System.out.println(a == c);    // true Integer与int比较时，a自动拆箱(调用a.intValue())成int类型再和c比较(实际上是两个int在比)
                               // 此处将3换为一个大于127的数结果不变。
```

```
Integer f1 = 100, f2 = 100, f3 = 150, f4 = 150;
System.out.println(f1 == f2);    // true 取常量池中的结果
System.out.println(f3 == f4);    // false new新对象
```

装箱的本质是什么呢？当我们给一个Integer对象赋一个int值的时候，javac编译器会调用Integer类的静态方法valueOf，如果整型字面量的值在[-128,127]之间，那么不会new新的Integer对象，而是直接引用常量池中的Integer对象，所以f1==f2的结果是true，而f3==f4的结果是false。

# char 型变量中能不能存贮一个中文汉字，为什么？

char类型可以存储一个中文汉字，因为Java中使用的编码是Unicode(不选择任何特定的编码，直接使用字符在字符集中的编号，这是统一的唯一方法)，一个char类型占2个字节(16比特)，所以放一个中文是没问题的。

使用Unicode意味着字符在JVM内部和外部有不同的表现形式，在JVM内部都是Unicode，当这个字符被从JVM内部转移到外部时(例如存入文件系统中)，需要进行编码转换。所以Java中有字节流和字符流，以及在字符流和字节流之间进行转换的转换流，如InputStreamReader和OutputStreamReader，这两个类是字节流和字符流之间的适配器类，承担了编码转换的任务。

# 例题:long

```
long a = 10000000000;    // 错误，超过了int类型长度
long b = 10000000000L;    // 正确
```

# 取余操作

浮点型也可取余，例如：

```
public class Test {
    public static void main(String[] args) {
        System.out.println(10.5 % 3);
        System.out.println(10.2 % 3);    // 精度丢失
    }
}
```

输出：

```
1.5
1.1999999999999993
```

Java中余数的计算公式为：

```
a%b=a-(a/b)*b
```

正数易于理解。举几个负数参与运算的小例子：

```
System.out.println(-5 % 3);    // -2
System.out.println(5 % -3);    // 2
System.out.println(-5 % -3);    // -2
```

小技巧：取余计算的结果的符号总与被除数相同，可以此验证计算结果的准确性。