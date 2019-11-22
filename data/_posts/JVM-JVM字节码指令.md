---
title: JVM-JVM字节码指令
date: 2017-11-21 15:26:49
tags: [Java,JVM]
categories: JVM
---

JVM的字节码指令由一个字节长度的，代表某种特定含义的数字(Opcode,即操作码)以及跟随其后的零至多个此操作所需的参数(Operands,即操作数)构成。由于JVM采用的是面向操作数栈而非寄存器的架构，因此大多数指令都仅有操作码，不包含操作数。

若不考虑异常处理，JVM的解释器执行字节码指令流的模型可用如下伪代码表示：

```
do {
    PC寄存器的字节码偏移量+1;
    根据PC寄存器指示的位置，从字节码指令流中取出操作码;
    if (该操作码需要操作数) 继续向后取出对应长度的操作数。PC寄存器随之后移;
    执行该字节码指令;
} while (剩余字节码指令流长度 > 0);
```

这里的+1是指加1个字节，即解释器读取字节码指令流的基本单位为1字节。若要读取一个16位的数据则需要连读两字节然后重新构建出原数据结构。例如可用byte1,byte2拼接存储short类型变量s。则解释器先读取byte1，后读取byte2。随后还原出s：

```
(byte1 << 8) | byte2
```

全部字节码指令集详见[JVM-JVM字节码指令集](/2017/11/16/JVM-JVM字节码指令集/)。

<!-- more -->

# 字节码与数据类型

JVM中绝大多数的字节码指令都限制了其操作的数据的数据类型(少数与数据类型无关的指令形如无条件跳转指令goto)。例如iload_0就是将局部变量表0号索引位置的int型数值压入操作数栈，而fload_0则是将局部变量表0号索引位置的float型数值压入操作数栈。这是严格的强类型检验，与解释器具体如何执行无关：即便最终解释器实际上是以同样的代码实现这两个指令，也不允许传入参数违法。

大部分与数据类型相关的字节码指令都会在其助记符中加入其所操作的类型的标记。例如i代表int，l代表long，s代表short，b代表byte，c代表char，f代表float，d代表double，a代表reference。也有少数字节码指令不会在其助记符中明示其所操作的数据类型，例如得到数组长度的arraylength指令，其后续必须跟随一个数组类型的对象。

与数据类型相关的字节码指令归纳如下：

![0.jpg](/images/blog_pic/JVM/JVM字节码指令/0.jpg)

![1.jpg](/images/blog_pic/JVM/JVM字节码指令/1.jpg)

![2.jpg](/images/blog_pic/JVM/JVM字节码指令/2.jpg)

很显然，并非所有数据类型都有对应的字节码指令。例如有iload却没有bload。事实上大多数字节码指令都不支持byte，char，short。而对于boolean则没有任何一种字节码指令支持。原因很简单，字节码指令的操作码只有1字节，最多也只能表示256种指令，为了使数据存储更为简洁就不得不在使用时下些功夫。事实上，编译器会在编译期或由JVM在运行期将byte及short类型的数据带符号扩展(Sign-Extend)为对应的int类型数据，而将boolean及char类型的数据零位扩展(Zero-Extend)为对应的int类型的数据。从而在处理byte,short,boolean,char类型的数据时可使用处理int类型数据的字节码指令。

事实上，如果我们看一下[JVM-类文件结构](/2017/11/07/JVM-类文件结构/)就可以发现，byte,short,boolean,char类型并未出现在类文件常量池的14种cp_info中，它们都被int类型所代替了。

不过，这样会导致Java在语法上不那么友好，以如下代码为例：

```
short s = 0;
s = s + 1;    // 无法通过编译
```

究其根本，还是进行加法时，s被自动扩展为了int型，其运算类型(Computational Type)变为了int，得到的结果自然也是int类型，当然就不能再赋值回short的变量了。

所有的字节码指令可大致分为后文中的9类。

# 加载和存储指令

加载和存储指令用于将数据在栈帧中的局部变量表和操作数栈之间传输。包括：

- 将局部变量表中的一个局部变量压入操作数栈：

```
iload,iload_<n>
lload,lload_<n>
fload,fload_<n>
dload,dload_<n>
aload,aload_<n>
```

- 将操作数栈的栈顶弹出并存入局部变量表：

```
istore,istore_<n>
lstore,lstore_<n>
fstore,fstore_<n>
dstore,dstore_<n>
astore,astore_<n>
```

- 将一个常量压入操作数栈：

```
bipush,sipush
ldc,ldc_w,ldc2_w
aconst_null
iconst_m1
iconst_<i>,lconst_<l>,fconst_<f>,dconst_<d>
```

- 扩充局部变量表的访问索引：wide

处理操作数栈及局部变量表中的数据主要就是通过上述指令。上述指令中有些带有&lt;n&gt;。例如iload_&lt;0&gt;等同于iload操作码后跟随int型的操作数0。之所以这么设计，是为了能让常用的数值的操作可以无需操作数而只以操作码完成，节省存储空间。当然指令是有限的(一共也就256个)，也不能无限的放宽常用值的范围。必须要取一个平衡。例如iload_&lt;n&gt;中n的取值范围为[0,3]。

# 运算指令

运算(算术)指令用于对操作数栈栈顶的一个或两个操作数进行某种特定的算术运算。其会将这一个或两个操作数弹出，运算后再将结果压入操作数栈。

大体上算术指令可分为如下两类：

- 对整型数据进行运算的指令
- 对浮点型数据进行运算的指令

处理整数和浮点数的指令在面对溢出和除零时会有不同的解决策略。但无论是哪种算术运算，都必须使用JVM指令集支持的数据类型。正如前文提到的，由于没有直接支持byte,short,boolean,char类型的算术运算指令，因此对于这些类型的算术运算，应该使用操作int类型的指令代替。

所有算术指令如下：

- 加法指令：iadd,ladd,fadd,dadd
- 减法指令：isub,lsub,fsub,dsub
- 乘法指令：imul,lmul,fmul,dmul
- 除法指令：idiv,ldiv,fdiv,ddiv
- 取余(模)指令：irem,lrem,frem,drem
- 取反指令：ineg,lneg,fneg,dneg
- 位移指令：ishl,lshl,ishr,lshr,iushr,lushr
- 按位或指令：ior,lor
- 按位与指令：iand,land
- 按位异或指令：ixor,lxor
- 局部变量自增指令：iinc
- 比较指令：lcmp,fcmpl,fcmpg,dcmpl,dcmpg

JVM字节码指令集直接支持了《Java语言规范》中描述的各种对整数及浮点数操作的语义。

关于整型数据，Java语法中在进行数据运算时可能会导致溢出，例如两个很大的正整数相加，结果可能是一个负数。但其实JVM规范中并没有明确规定整型数据溢出的运算规则，仅规定了在处理整型数据时，除了除法指令(idiv,ldiv)及取余(模)指令(irem,lrem)会因除数为零抛出ArithmeticException异常外，其余任何整型运算场景都不会抛出运行时异常。JVM规范的限制就这么多，只要满足了这个限制，具体的溢出规则就由具体实现自行把控。

关于浮点型数据的运算，JVM规范的限制明显大了很多：运算必须严格遵循IEEE754规范所规定的行为及限制。具体来说，必须严格遵守IEEE754规范所规定的非正规浮点数值(Denormalized Floating-Point Numbers)及逐级下溢(Gradual Underflow)的运算规则。

在进行浮点数运算时，所有运算结果必须舍入到适当的精度。舍入模式为IEEE754规范中默认的舍入模式。即向最接近数舍入模式：非精确的结果必须舍入为可被表示的最接近的精确值，若上下两方精确值同样接近，则优先选择最低有效位为零的。

在将浮点数转换为整数时，采用IEEE754规范中的向零舍入模式：数字会被截断，所有小数部分均会被舍弃。例如55.2舍入结果为55。-55.2舍入结果为-55(注意，不是-56。舍入为粗暴的舍弃小数部分，而非找到不大于原值的最大整数)。

和整型运算不同的是，JVM规范要求浮点数运算时不会抛出任何运行时异常(这里的异常是指Java层面的异常，不是IEEE754规范中的作为运算信号的浮点异常)。当浮点运算溢出时，将使用有符号的无穷大表示。若某运算结果没有明确的数学定义，将使用NaN表示。而所有NaN参与的运算结果均返回NaN。

比较long型数据时(lcmp)，采用的是带符号的比较方式。而比较浮点型数据时(fcmpl,fcmpg,dcmpl,dcmpg)，采用的是IEEE754规范所定义的无信号比较(Nonsignaling Comparisons)方式。

# 类型转换指令

类型转换指令可以将两种不同的数值类型进行相互转换，其用途主要有2：

- 源码中显式指定的强制类型转换
- 前文提到的针对char,boolean,byte,short这4种类型的自动扩展

类型转换主要分为两类，一类是JVM直接支持的，转换时无需显式转换指令的，被称为宽化的类型转换(Widening Numeric Conversions)，一如其名，即小范围类型向大范围类型的安全转换：

- int --> long或float或double
- long --> float或double
- float --> double

相对的，另一类类型转换被称为窄化类型转换(Narrowing Numeric Conversions)，一如其名，即大范围类型向小范围类型的不安全转换。必须显式的使用转换指令完成。这些转换指令包括：

- i2c,i2b,i2s
- l2i
- f2l,f2i
- d2f,d2l,d2i

窄化类型转换可能会导致转换结果正负号较之原值发生变化。不同数量级的情况还可能导致精度丢失。

在对long或int做窄化类型转换时，转换的过程仅仅是保留接收类型长度的最低几位。因符号位位于最高位，必然导致符号位被截取掉，从而可能使符号发生变化(之所以说是可能，是因为截断后的最高位可能仍与原值相同)。

在将一个浮点数窄化为一个整数(特指long,int)时，将遵循以下规则：

- 若浮点值是NaN，那转换结果就是int或long类型的0
- 若浮点值不是无穷大的话，将采用IEEE754规范中的向零舍入模式：数字会被截断，所有小数部分均会被舍弃，获得整数值v。若v在接收类型能表示的范围之内，那转换结果就是v
- 反之，将根据v的符号，表示为接收类型能表示的最大正数或最小负数。

而浮点数内部的窄化，即double向float的转换，遵循IEEE754规范中的向最接近数舍入模式，得到一个float精度能表示的值。若舍入结果的绝对值过小超出了float的精度表示范围，将返回float类型的正负零。如果返回的绝对值过大超出了float的表示范围，将返回float类型的正负无穷大。double类型的NaN将转换为float类型的NaN。

尽管如上文所述，数据类型窄化可能会导致下限溢出，上限溢出或精度丢失。但是JVM规范明确规定不能因数据类型窄化而抛出运行时异常。

# 对象创建与访问指令

虽然从理论层面来讲，Java中的类实例与数组实例都是对象。但是在实现上，JVM对类实例及数组实例的创建与访问使用了不同的字节码指令。

具体指令如下：

- 创建类实例的指令：new
- 创建数组实例的指令：newarray,anewarray,multianewarray
- 访问类字段(即类变量，被static修饰的字段)和实例字段(即实例变量，未被static修饰的字段)的指令：getstatic,putstatic,getfield,putfield
- 把一个数组元素压入操作数栈的指令：caload,baload,saload,iaload,laload,faload,daload,aaload
- 把操作数栈栈顶的值弹出作为数组元素存储入数组的指令：castore,bastore,sastore,iastore,lastore,fastore,dastore,aastore
- 取数组长度的指令：arraylength
- 检查类实例类型的指令：instanceof,checkcast

# 操作数栈管理指令

如同操作一个普通的栈型数据结构那样，JVM提供了一些指令直接操作操作数栈：

- 将操作数栈栈顶的一个或两个元素出栈的指令：pop,pop2
- 复制操作数栈栈顶一个或两个元素并将这一个或两个元素的复制值重新压入操作数栈的指令：dup,dup2,dup_x1,dup2_x1,dup_x2,dup2_x2
- 将操作数栈最顶端的两个数值交换的指令：swap

# 控制转移指令

控制转移指令可以让解释器有条件或无条件的从指定位置的指令而非该控制转移指令的下一条指令继续执行。从概念模型上理解，可以认为控制转移指令就是在有条件或无条件的修改PC寄存器中的值。控制转移指令包括：

- 条件分支指令：ifeq,iflt,ifle,ifne,ifgt,ifge,ifnull,ifnonnull,if_icmpeq,if_icmpne,if_icmplt,if_icmpgt,if_icmple,if_icmpge,if_acmpeq,if_acmpne
- 复合条件分支指令：tableswitch,lookupswitch
- 无条件跳转指令：goto,goto_w,jsr,jsr_w,ret

如上文所示，JVM中有专门的处理int及引用类型的条件分支比较指令。为了便于处理null这一特殊值，也会为其设置专门的检测指令。

同前文规则一致，对于boolean,char,byte,short型数据的条件分支比较，均采用int型的比较指令完成。

对于long,float,double而言，会先用运算指令中的比较指令：lcmp,fcmpl,fcmpg,dcmpl,dcmpg得到[1,0,-1]这样int型的比较结果并存入操作数栈，随后再用int型条件分支指令处理这些结果值，从而完成整个条件分支跳转。

由上文描述可知，所有基本数据类型的条件分支跳转实际上最终都依托于int型的条件分支跳转，更进一步的说，是依托于int型的比较操作。因此int型的比较操作是否简便完善高效就显得尤为重要。这也是JVM字节码指令集中int型的比较指令最为丰富强大的原因。

# 方法调用和返回指令

与方法调用(即分派+执行)相关的指令列举如下：

- invokevirtual：调用实例方法。会根据对象的实际类型进行动态单分派(虚方法分派)
- invokeinterface：调用接口方法。运行期解释器会搜索一个实现了该接口方法的对象，并调用对应实现的接口方法
- invokespecial:以操作数栈栈顶reference类型的数据所指向的对象为方法的接收者，调用此对象的实例构造器&lt;init&gt;方法，私有方法或超类构造方法。该指令的操作码之后会紧跟一个u2的操作数说明具体调用的是哪个方法，该参数指向常量池集合中的一个CONSTANT_UTF8_info类型的索引项，也就是该方法的方法符号引用。
- invokestatic：调用类方法(static修饰的方法)。
- invokedynamic：至JDK1.7为止仍未存在于JVM字节码指令集中。用于在运行时动态解析出调用点限定符所引用的方法，并执行该方法。前面介绍的4条方法调用指令的分派逻辑都固化在JVM内部，而本指令的分派逻辑则由用户设定的引导方法决定。

方法返回指令是根据返回值的类型区分的，包括:

- ireturn(自然，延续前文的规定，除了int之外，boolean,char,byte,short也使用该指令)
- lreturn
- freturn
- dreturn
- areturn
- return：供返回值为void的方法或实例初始化方法或类及接口的类初始化方法使用。

# 异常处理指令

显式异常指令(源代码中明确用throw语句标记的异常)均由athrow指令实现。此外，其他字节码指令在检测到异常状况时也会自动抛出异常，例如前文介绍过的除法指令(idiv,ldiv)及取余(模)指令(irem,lrem)会因除数为零抛出ArithmeticException异常。

而源码中显式被catch语句捕获的异常则不是由字节码指令实现的(此前是用jsr及ret指令实现的，但在JDK1.7中这两条指令已废弃了)，而是采用了class文件->方法表集合->方法表->属性集合->Code属性->异常表集合实现的。

# 同步指令

主要为synchronized关键字服务，详见[Java 并发-synchronized](/2017/07/13/Java 并发-synchronized/)。