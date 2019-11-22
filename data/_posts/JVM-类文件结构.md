---
title: JVM-类文件结构
date: 2017-11-07 16:59:49
tags: [Java,JVM]
categories: JVM
---

任意一个class文件都对应一个唯一的类或接口。但类或接口并非必须定义在class文件中(也可通过类加载器在运行期直接生成)。本文所指的类文件结构是指满足规范的任意类或接口的class结构。

class文件是一个以1字节(即8位)为基础的二进制流。各数据项依规范紧密有序的排列着，没有分隔符的存在。当数据项需占用1字节以上的空间时，将采用高位在前的方式切割为若干个1字节的基本单位。

以十进制为例：

- 高位在前(Big-Endian)：123代表一百二十三。这也是人类的习惯。
- 低位在前(Little-Endian)：123代表三百二十一。

<!-- more -->

class文件的数据结构是一种类似于C语言结构体的伪结构，这种伪结构中只有两种数据类型：无符号数及表。

无符号数是基本的数据类型。以u1,u2,u4,u8分别代表1个字节，2个字节，4个字节，8个字节。无符号数可用来描述数字，索引引用，数量值或按照UTF-8缩略编码的字符串。

表是由无符号数或其它表作为数据项构成的复合数据类型。通常表习惯性的以_info结尾。

整个class文件本质上就是一张表。其数据项为：

![0.jpg](/images/blog_pic/JVM/类文件结构/0.jpg)

若某数据类型的个数不定，通常会在最前面加上计数。后文将计数及对应个数个相同的数据类型作为一个整体(集合)描述。

# magic

class文件的头4个字节被称为魔数(Magic Number)。其唯一的作用为确认该文件是否是一个身份合法的，能被JVM读取的class文件。

其实不仅仅是class，很多文件格式(例如gif,jpeg等)都采用魔数进行身份验证，其较之文件扩展名安全性更高(相对来说，扩展名更易于被改变)。魔数值只要保证在文件使用的范围内唯一即可。

示例代码：

```
public class Test {
}
```

以16进制编辑器打开其class文件：

![1.jpg](/images/blog_pic/JVM/类文件结构/1.jpg)

class文件的魔数用16进制表示为：CAFEBABE(即咖啡宝贝。能只用A~F6个字母拼出一个萌萌哒且和Java语义相关的魔数也是碉堡了)

# minor_version+major_version

紧接着魔数之后的4个字节为class文件的版本号。其中前2个字节为次版本号(minor_version)，后两个字节为主版本号(major_version)。

主版本号始于45。即：

- JDK1.1=45
- JDK1.2=46
- JDK1.3=47
- JDK1.4=48
- JDK1.5=49
- JDK1.6=50
- JDK1.7=51
- JDK1.8=52

依此类推。

高版本的JDK可兼容运行低版本的class文件，反之则不行：即使两个JDK版本间的class文件格式并未发生变化也不行。

示例代码：

```
public class Test {
}
```

以JDK1.7编译：

![2.jpg](/images/blog_pic/JVM/类文件结构/2.jpg)

其中0x33=51，即为JDK1.7。

# 常量池集合

主版本号之后为常量池集合。常量池可以看作是class文件的资源仓库，是class文件结构中与class的其他项目关联最多的数据类型，也是占用class空间最大的数据项之一。

示例代码：

```
public class Test {

    private int m;

    public int inc() {
        return m + 1;
    }
}
```

以JDK1.7编译：

![3.jpg](/images/blog_pic/JVM/类文件结构/3.jpg)

常量池集合分为两部分：第一部分为两个字节，表示常量池大小(constant_pool_count)。第二部分为constant_pool_count-1个常量类型(cp_info)。

常量池集合的第0项被空了出来：表示"不引用任何一个常量池项目"。只有常量池集合有这个特殊的设定，其它的集合(接口索引集合，字段表集合，方法表集合等)均按常规从0开始存放数据项。因此，实际有效的常量数据项是索引1开始的，constant_pool_count也比cp_info类型的个数多一个。

![4.jpg](/images/blog_pic/JVM/类文件结构/4.jpg)

如上图所示，示例程序constant_pool_count=0x13，即为十进制的19。即共有18个常量类型。

常量类型主要分为两大类：字面量(Literal)和符号引用(Symbolic References)。

字面量类似于Java语法层面的常量值概念。如文本字符串等。

符号引用则是编译原理方面的概念，包含以下3类：

- 类和接口的全限定名(Fully Qualified Name)
- 字段的名称和描述符(Descriptor)
- 方法的名称和描述符

C与C++的编译是一步到位的，编译的结果直接就是物理机器运行的二进制机器码。因此编译时会有“连接”这一步骤：即将代码中的方法，字段等与实际的物理地址相关联。而Java的编译仅仅只是编译为供JVM使用的中间字节码，并没有连接这一操作，真正的关联方法字段与其实际物理地址的操作是在运行期完成的，称为动态连接。因此，编译出的class文件中方法及字段是以符号引用的方式存储的：相当于一个指引，在运行期告诉JVM这是个什么东西，该翻译到什么内存地址中。

cp_info只是一个总称，实际上又可分为很多种。每种cp_info都是一张表。在JDK1.7之前，共有11种cp_info。JDK1.7时为了更好的支持动态语言调用，又添加了以下3种：

- CONSTANT_MethodHandle_info
- CONSTANT_MethodType_info
- CONSTANT_InvokeDynamic_info

14种cp_info如下图所示：

![5.jpg](/images/blog_pic/JVM/类文件结构/5.jpg)

这14种cp_info的共同特点为开头均为1个字节的类型标志位(tag，即上图中的标志)，而后才是该种cp_info的具体值。这样根据每种cp_info的规范就可严格限定出该种cp_info所占用的空间。进而严格限定出常量池占用的空间。

14种常量类型的具体结构依如下3图所示：

![6.jpg](/images/blog_pic/JVM/类文件结构/6.jpg)

![7.jpg](/images/blog_pic/JVM/类文件结构/7.jpg)

![8.jpg](/images/blog_pic/JVM/类文件结构/8.jpg)

下面具体分析下示例代码中的常量池集合：

**1**

首先，跟在constant_pool_count=0x13=19之后的第一个常量为：

![9.jpg](/images/blog_pic/JVM/类文件结构/9.jpg)

0x0A=10，查表为CONSTANT_Methodref_info，即为类中方法的符号引用。其数据结构共占据4个字节：

![10.jpg](/images/blog_pic/JVM/类文件结构/10.jpg)

- 前2个字节，0x0004=4，指向类的CONSTANT_Class_info的索引项
- 后2个字节，0x000F=15，指向类的CONSTANT_NameAndType_info的索引项

---

**2**

第2个常量：

![11.jpg](/images/blog_pic/JVM/类文件结构/11.jpg)

0x09=9，查表为CONSTANT_Fieldref_info，即字段的符号引用。其数据结构共占据4个字节：

![12.jpg](/images/blog_pic/JVM/类文件结构/12.jpg)

- 前2个字节，0x0003=3，指向字段所属的CONSTANT_Class_info的索引项
- 后2个字节，0x0010=16，指向字段的CONSTANT_NameAndType_info的索引项

---

**3**

第3个常量：

![13.jpg](/images/blog_pic/JVM/类文件结构/13.jpg)

0x07=7，查表为CONSTANT_Class_info，即类或接口的符号引用。其数据结构共占据2个字节：

![14.jpg](/images/blog_pic/JVM/类文件结构/14.jpg)

0x0011=17。指向一个CONSTANT_UTF8_info类型的常量，代表类或接口的全限定名。

---

**4**

第4个常量：

![15.jpg](/images/blog_pic/JVM/类文件结构/15.jpg)

0x07=7，查表为CONSTANT_Class_info，即类或接口的符号引用。其数据结构共占据2个字节：

![16.jpg](/images/blog_pic/JVM/类文件结构/16.jpg)

0x0012=18。指向一个CONSTANT_UTF8_info类型的常量，代表类或接口的全限定名。

---

**5**

第5个常量：

![17.jpg](/images/blog_pic/JVM/类文件结构/17.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。

与普通UTF-8编码所不同，UTF-8压缩编码的编码方式为：

- ['\u0001','\u007f']之间的字符(即[1,127]之间的ASCII码)用1个字节表示
- ['\u0080','\u07ff']之间的字符用2个字节表示
- ['\u0800','\uffff']之间的字符和普通的UTF-8编码规则相同，用3个字节表示

class文件中类及接口的全路径名，方法名，变量名均由CONSTANT_UTF8_info描述。而描述该长度的值占两个字节。因此可表示的范围为[0,65535]。即能表示的字节长度最多为65535个(若字符均在ASCII码的范围之内，也可说最多表示65535个字符)。

CONSTANT_UTF8_info的数据结构占据的长度不定，tag后是表示长度的2个字节，代表字符串占用的字节数：

![18.jpg](/images/blog_pic/JVM/类文件结构/18.jpg)

0x0001=1，即字符串占用1个字节，则向后取一个字节：

![19.jpg](/images/blog_pic/JVM/类文件结构/19.jpg)

0x6D=109。即为ASCII编码的"m"。

---

**6**

第6个常量：

![20.jpg](/images/blog_pic/JVM/类文件结构/20.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![21.jpg](/images/blog_pic/JVM/类文件结构/21.jpg)

0x0001=1，即字符串占用1个字节，则向后取一个字节：

![22.jpg](/images/blog_pic/JVM/类文件结构/22.jpg)

0x49=73。即为ASCII编码的"I"。

---

**7**

第7个常量：

![23.jpg](/images/blog_pic/JVM/类文件结构/23.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![24.jpg](/images/blog_pic/JVM/类文件结构/24.jpg)

0x0006=6，即字符串占用6个字节，则向后取6个字节：

![25.jpg](/images/blog_pic/JVM/类文件结构/25.jpg)

---

**8**

第8个常量：

![26.jpg](/images/blog_pic/JVM/类文件结构/26.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![27.jpg](/images/blog_pic/JVM/类文件结构/27.jpg)

0x0003=3，即字符串占用3个字节，则向后取3个字节：

![28.jpg](/images/blog_pic/JVM/类文件结构/28.jpg)

---

**9**

第9个常量：

![29.jpg](/images/blog_pic/JVM/类文件结构/29.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![30.jpg](/images/blog_pic/JVM/类文件结构/30.jpg)

0x0004=4，即字符串占用4个字节，则向后取4个字节：

![31.jpg](/images/blog_pic/JVM/类文件结构/31.jpg)

---

**10**

第10个常量：

![32.jpg](/images/blog_pic/JVM/类文件结构/32.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![33.jpg](/images/blog_pic/JVM/类文件结构/33.jpg)

0x000F=15，即字符串占用15个字节，则向后取15个字节：

![34.jpg](/images/blog_pic/JVM/类文件结构/34.jpg)

---

**11**

第11个常量：

![35.jpg](/images/blog_pic/JVM/类文件结构/35.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![36.jpg](/images/blog_pic/JVM/类文件结构/36.jpg)

0x0003=3，即字符串占用3个字节，则向后取3个字节：

![37.jpg](/images/blog_pic/JVM/类文件结构/37.jpg)

---

**12**

第12个常量：

![38.jpg](/images/blog_pic/JVM/类文件结构/38.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![39.jpg](/images/blog_pic/JVM/类文件结构/39.jpg)

0x0003=3，即字符串占用3个字节，则向后取3个字节：

![40.jpg](/images/blog_pic/JVM/类文件结构/40.jpg)

---

**13**

第13个常量：

![41.jpg](/images/blog_pic/JVM/类文件结构/41.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![42.jpg](/images/blog_pic/JVM/类文件结构/42.jpg)

0x000A=10，即字符串占用10个字节，则向后取10个字节：

![43.jpg](/images/blog_pic/JVM/类文件结构/43.jpg)

---

**14**

第14个常量：

![44.jpg](/images/blog_pic/JVM/类文件结构/44.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![45.jpg](/images/blog_pic/JVM/类文件结构/45.jpg)

0x0009=9，即字符串占用9个字节，则向后取9个字节：

![46.jpg](/images/blog_pic/JVM/类文件结构/46.jpg)

---

**15**

第15个常量：

![47.jpg](/images/blog_pic/JVM/类文件结构/47.jpg)

0x0C=12，查表为CONSTANT_NameAndType_info，即字段或方法的描述信息。其数据结构共占据4个字节：

![48.jpg](/images/blog_pic/JVM/类文件结构/48.jpg)

- 前2个字节，0x0007=7，指向该字段或方法名称的CONSTANT_UTF8_info的索引项(name)
- 后2个字节，0x0008=8，指向该字段或方法描述符的CONSTANT_UTF8_info的索引项(type)

---

**16**

第16个常量：

![49.jpg](/images/blog_pic/JVM/类文件结构/49.jpg)

0x0C=12，查表为CONSTANT_NameAndType_info，即字段或方法的描述信息。其数据结构共占据4个字节：

![50.jpg](/images/blog_pic/JVM/类文件结构/50.jpg)

- 前2个字节，0x0005=5，指向该字段或方法名称的CONSTANT_UTF8_info的索引项(name)
- 后2个字节，0x0006=6，指向该字段或方法描述符的CONSTANT_UTF8_info的索引项(type)

---

**17**

第17个常量：

![51.jpg](/images/blog_pic/JVM/类文件结构/51.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![52.jpg](/images/blog_pic/JVM/类文件结构/52.jpg)

0x000D=13，即字符串占用13个字节，则向后取13个字节：

![53.jpg](/images/blog_pic/JVM/类文件结构/53.jpg)

---

**18**

第18个常量：

![54.jpg](/images/blog_pic/JVM/类文件结构/54.jpg)

0x01=1，查表为CONSTANT_UTF8_info，即用UTF-8压缩编码的字符串。其长度为：

![55.jpg](/images/blog_pic/JVM/类文件结构/55.jpg)

0x0010=16，即字符串占用16个字节，则向后取16个字节：

![56.jpg](/images/blog_pic/JVM/类文件结构/56.jpg)

---

**javap**

至此，18个常量全部分析完成。实际上，在jdk/bin中提供了工具javap用来分析class文件：

![57.jpg](/images/blog_pic/JVM/类文件结构/57.jpg)

分析文件1内容如下：

```
Classfile /E:/Test.class
  Last modified 2017-11-8; size 274 bytes
  MD5 checksum ba2585a36b64eb15c6657dc8440c38e0
  Compiled from "Test.java"
public class com.test.Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         //  com/test/Test.m:I
   #3 = Class              #17            //  com/test/Test
   #4 = Class              #18            //  java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               Test.java
  #15 = NameAndType        #7:#8          //  "<init>":()V
  #16 = NameAndType        #5:#6          //  m:I
  #17 = Utf8               com/test/Test
  #18 = Utf8               java/lang/Object
{
  public com.test.Test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 3: 0

  public int inc();
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0       
         1: getfield      #2                  // Field m:I
         4: iconst_1      
         5: iadd          
         6: ireturn       
      LineNumberTable:
        line 8: 0
}
```

观察常量池Constant pool部分，其内容与上文的分析完全一致。

在作为示例的Test类中，除了我们显式定义的字段m，方法inc()之外，常量池中还有诸如

```
<init>
()V
Code
LineNumberTable
SourceFile
```

等我们没有显式声明的有些莫名其妙的东西。这些常量会在内部中间逻辑中被常量池集合本身，字段表(field_info)，方法表(method_info)，属性表(attribute_info)等用到，对用户而言是透明的。它们被用以形容那些不方便用固定字节表示的信息(其实就是更接近于人类自然语言表述的信息)。

# access_flags

紧接着常量池集合之后的两个字节为访问标志(access_flags)。用于标志一些类或接口层次的访问信息：

![58.jpg](/images/blog_pic/JVM/类文件结构/58.jpg)

2个字节共计16位。因此共可设置16种标志。目前只使用了其中的8种(JDK1.5以前只有上图中的前5种，JDK1.5中又引入了后3种)。没有使用的标志默认为0。

仍然使用常量池集合的代码为例。Test类符合ACC_PUBLIC,ACC_SUPER。即：

ACC_PUBLIC=0x0001=0000_0000_0000_0001
ACC_SUPER=0x0020=0000_0000_0010_0000
则最终为0000_0000_0010_0001=0x0021。即：

![59.jpg](/images/blog_pic/JVM/类文件结构/59.jpg)

# this_class

紧接着访问标志之后的2个字节是类索引(this_class)，用以描述该类的全限定名。其指向常量池集合中的一个CONSTANT_Class_info类型的索引项。

仍然使用常量池集合的代码为例：

![60.jpg](/images/blog_pic/JVM/类文件结构/60.jpg)

0x0003=3。即为com/test/Test。

# super_class

紧接着类索引之后的2个字节是父类索引(super_class)。用以描述该类父类的全限定名。其指向常量池集合中的一个CONSTANT_Class_info类型的索引项。

因为Java为单继承，因此父类索引仅需1个。又由于所有类都继承自Object，因此除Object之外，其他类的父类索引均不为0。而Object类的父类索引为0，表示没有父类。

仍然使用常量池集合的代码为例：

![61.jpg](/images/blog_pic/JVM/类文件结构/61.jpg)

0x0004=4。即为java/lang/Object。

# 接口集合

紧接着父类索引之后的是接口集合。用来描述该类实现了哪些接口。其内部首先会用2个字节记录implements(若class表示的是接口则是extends)的接口数量interfaces_count。而后是interfaces_count个长度为2字节的interface。每个interface都指向常量池集合中的一个CONSTANT_Class_info类型的索引项。interface的排列顺序与代码中的声明顺序相同。

仍然使用常量池集合的代码为例，Test没有实现接口，即interfaces_count=0，后续没有interface数据项：

![62.jpg](/images/blog_pic/JVM/类文件结构/62.jpg)

# 字段表集合

紧接着接口集合之后的是字段表集合(field_info)，用以描述类或接口中声明的变量。

其内部分为两部分：第一部分为2个字节，表示字段个数(fields_count)。第二部分为fields_count个字段类型(field_info)。

仍然使用常量池集合的代码为例，其fields_count=1：

![63.jpg](/images/blog_pic/JVM/类文件结构/63.jpg)

field_info格式如下(排列顺序为先左后右)：

![64.jpg](/images/blog_pic/JVM/类文件结构/64.jpg)

示例代码的字段表集合中只有一个field_info，其结构列举如下：

**access_flags**

field_info的第一部分占用2字节，其作用及设置方式与类或接口的access_flags非常类似，都是通过设置标志位：

![65.jpg](/images/blog_pic/JVM/类文件结构/65.jpg)

2字节可设置16种标志，只使用了其中的9种。未使用的默认为0。

很显然，ACC_PUBLIC,ACC_PRIVATE,ACC_PROTECTED最多只能3选1，ACC_FINAL,ACC_VOLATILE最多只能2选1。而若class文件表示的是接口，则必须有ACC_PUBLIC,ACC_STATIC,ACC_FINAL。

仍然使用常量池集合的代码为例：

![66.jpg](/images/blog_pic/JVM/类文件结构/66.jpg)

0x0002=0000-0000-0000-0010。即只设置了ACC_PRIVATE。

---

**name_index**

紧跟着access_flags之后的2字节是name_index，代表字段的简单名称。指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项。

仍然使用常量池集合的代码为例：

![67.jpg](/images/blog_pic/JVM/类文件结构/67.jpg)

0x0005=5，即为m。

---

**descriptor_index**

用于描述字段及方法的属性。主要有3个：

- 全限定名
- 简单名称
- 描述符

全限定名和简单名称较易于理解，以示例代码为例，类Test的全限定名为com/test/Test。方法inc()的简单名称为inc，字段m的简单名称为m。

描述符所表述的信息则更为复杂。对于字段而言，描述的是字段的数据结构。对于方法而言，描述的是方法的参数列表(数量，类型，顺序)和返回值。

根据描述符规范，基本数据类型(byte,char,double,float,int,long,short,boolean)以及代表无返回值的void(JVM规范将void单独列出，名为VoidDescriptor)都用一个大写字母表示。引用类型则使用大写字母L加对象的全限定名表示：

![68.jpg](/images/blog_pic/JVM/类文件结构/68.jpg)

因引用类型长度不定，故仅有其最后会加上一个;以示结束。

对于数组类型而言，每有一个维度，都将增加一个前置的[。例如：

- java.lang.String[][]表示为[[Ljava/lang/String;
- int[]表示为[I

描述方法时，按照先参数列表后返回值的顺序记录。参数列表按参数顺序写入一个()中：

```
void m1()
描述为：
()V
```

```
int m2(char[] c, int i1, int i2, String[][] s, double d)
描述为：
([CII[[Ljava/lang/String;D)I
```

仍然使用常量池集合的代码为例：

![69.jpg](/images/blog_pic/JVM/类文件结构/69.jpg)

0x0006=6。即为I。至此综合access_flags，name_index及descriptor_index。我们已可记录该字段的基本信息：private int m。

---

**属性表集合**

descriptor_index之后是属性表集合。其内部分为两部分：首先是u2长度的attributes_count，随后是attributes_count个attribute_info类型。上文已介绍的access_flags，name_index及descriptor_index是field_info必定包含的数据，而属性表集合则是可选数据，若不存在则有attributes_count=0，属性表集合直接结束。仍然使用常量池集合的代码为例：

![70.jpg](/images/blog_pic/JVM/类文件结构/70.jpg)

此时attributes_count=0x0000=0，即不包含属性表。

属性表集合用于存储一些额外信息。例如有类变量：

```
final static int f = 123;
```

则其属性表集合中可能会存在一项名为ConstantValue的属性，其值指向常量池集合中的常量123(详见后文对属性表集合的介绍)。

---

**一些细节**

字段表集合中不会列出从父类(对于类而言)或父接口(对于接口而言)中继承而来的字段，但是可能会列出没有在代码中显式声明的字段：例如内部类为了保持对外部类的访问性，会自动添加指向外部类实例的字段。

对于Java语言规范而言，字段是无法重载的，两个字段的数据类型，修饰符不管是否相同，都必须使用不同的字段名。而对于JVM规范而言，只要field_info不同即可，换句话说，字段可以重名。

# 方法表集合

紧跟在字段表集合之后的是方法表集合。方法表集合的存储思路与字段表集合几乎完全相同。

其内部分为两部分：第一部分为2个字节，表示方法个数(methods_count)。第二部分为methods_count个方法类型(method_info)。

仍然使用常量池集合的代码为例，其methods_count=2：

![71.jpg](/images/blog_pic/JVM/类文件结构/71.jpg)

method_info格式如下(排列顺序为先左后右)：

![72.jpg](/images/blog_pic/JVM/类文件结构/72.jpg)

示例代码的方法表集合中有2个method_info，现以第一个为例：

**access_flags**

method_info的第一部分占用2字节，其存储思路类似于field_info的access_flags：

![73.jpg](/images/blog_pic/JVM/类文件结构/73.jpg)

因字段与方法修饰符的差异，method_info较之field_info在access_flags上做了如下增删：

- 删除只能修饰字段不能修饰方法的：volatile(ACC_VOLATILE),transient(ACC_TRANSIENT)
- 增加只能修饰方法不能修饰字段的：synchronized(ACC_SYNCHRONIZED),native(ACC_NATIVE),strictfp(ACC_STRICTFP，即FP-strict，精确浮点计算),abstract(ACC_ABSTRACT)

则第一个方法的access_flags为：

![74.jpg](/images/blog_pic/JVM/类文件结构/74.jpg)

0x0001=0000-0000-0000-0001。即只设置了ACC_PUBLIC。

---

**name_index**

紧跟着access_flags之后的2字节是name_index，代表方法的简单名称。指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项。

则第一个方法的name_index为：

![75.jpg](/images/blog_pic/JVM/类文件结构/75.jpg)

0x0007=7，即为&lt;init&gt;。

---

**descriptor_index**

类似于field_info的descriptor_index，方法1的descriptor_index为：

![76.jpg](/images/blog_pic/JVM/类文件结构/76.jpg)

0x0008=8。即为()V。至此综合access_flags，name_index及descriptor_index。我们已可记录该方法的基本信息：

```
public void <init>();
```

该方法并未在代码中显式声明，是由编译器自动添加的实例构造器。

---

**属性表集合**

descriptor_index之后是属性表集合。类似于field_info的属性表集合，其内部分为两部分：首先是u2长度的attributes_count，随后是attributes_count个attribute_info类型。上文已介绍的access_flags，name_index及descriptor_index是method_info必定包含的数据，而属性表集合则是可选数据，若不存在则有attributes_count=0，属性表集合直接结束。则对于方法1而言：

![77.jpg](/images/blog_pic/JVM/类文件结构/77.jpg)

此时attributes_count=0x0001=1，即包含1个属性表。

关于属性表信息，详见后文对属性表集合的介绍。

---

**一些细节**

若没有发生重写(Override)，则子类class文件中的方法表集合中不会出现父类的方法。实际上，如果我们换个思路来思考：和字段表集合一样，方法表集合中同样不会出现父类方法。所谓的重写不过是子类中的方法覆盖了父类中的方法，该方法实际上仍是属于子类的方法，并非父类中的那个方法。

在[Java 基础-重载与重写](/2017/10/10/Java 基础-重载与重写/)中，规定了重载的判定依据为：

- **方法的参数**：必须一模一样，包括个数，顺序，类型(有继承关系的子类也不行，必须是一模一样的类)
- **返回值**：一样或为有继承关系的子类
- **异常检查**：对于Checked Exception而言，可以抛出更少的异常，但不能抛出父类中没有定义的异常。对于Unchecked Exception(RuntimeException)及Error而言则没有限制。
- **访问权限**：应比父类中的权限更宽松，换句话说，即允许被更多人访问(public > protected > default[即没有修饰] > private)。

之所以对方法参数的要求如此严格，就是因为要严格遵循method_info中的descriptor_index中的()中的参数列表值。

类似于字段表集合，方法表集合中也可能会出现没有在代码中显式定义的方法。最典型的就是类构造器方法&lt;clinit&gt;及实例构造器方法&lt;init&gt;。

如前所述，在Java语言规范中，字段是无法重载的。在JVM规范中，字段是可以重载的。而Java语言规范及JVM规范中方法均是可以重载的，重载的前提条件为方法间的简单名称相同，而特征签名不同。

Java语言规范与JVM规范对于特征签名的定义是不同的：

- Java语言规范：包括方法名称，参数顺序，参数类型(不考虑子类，即参数分别为父类及子类的两个方法间可以发生重载)。
- JVM规范：在Java语言规范的基础上，又添加了方法返回值及受查异常表(均不考虑子类)。

通常我们如果只说特征签名，那么指的就是Java语言规范中的特征签名。

类似于字段的重载，由两个规范对特征签名定义的差异可知，在Java语言规范中无法共存的两个方法在class文件中就有可能共存(举个具体的例子，两个方法仅仅依靠返回值类型不同就可以在class文件中共存，却无法通过Java语法的编译)。

# 属性表集合

属性表(attribute_info)集合是class文件中最灵活(相对来说)也是最重要的一部分。其出现在4处地方：

- 紧跟在方法表集合之后(即整个类文件的最后)，用于描述类的额外信息
- 每个字段表的最后，用于描述字段的额外信息
- 每个方法表的最后，用于描述方法的额外信息，其中最为重要的一种attribute_info为Code，用于记录方法体内部的代码
- 部分attribute_info(例如Code)内部的最后也会包含

由此可以归纳出class文件的设计套路：每个结构先用尽量工整死板的结构存储必要的信息，然后将较为灵活的信息作为额外数据放在最后。

之所以说属性表最为灵活，是因为不再要求属性表集合中的attribute_info有固定的顺序。甚至只要不与已有的属性表重名，任何人实现的任何编译器都可以向属性表集合中写入自己定义的attribute_info。JVM在运行期若遇到不认识的attribute_info会自动忽略。

属性表集合内部分为两部分：首先是u2长度的attributes_count，随后是attributes_count个attribute_info类型。

每个attribute_info的结构为：

![78.jpg](/images/blog_pic/JVM/类文件结构/78.jpg)

首先是attribute_name_index，占2个字节，它指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项。用于表示attribute_info的名称，也是attribute_info间区分的依据(前文所谓的重名)。随后是attribute_length，占4个字节，用于描述该attribute_info的净长度(即不算attribute_name_index与attribute_length，实际存储数据信息的长度)。最后就是占用attribute_length*u1空间的属性信息本身。

由该定义我们可以再次认识到，属性表的定义确实很灵活。只需要告诉JVM你所要记录的属性信息有多长，给一个从外部划分的边界，至于内容则是黑盒的，JVM规范并未规定。

JVM规范自带的attribute_info如下所示。最初在《Java虚拟机规范(第二版)》中预定义了9种attribute_info，而到了《Java虚拟机规范(Java SE 7)》中，attribute_info扩展到了21种：

![79.jpg](/images/blog_pic/JVM/类文件结构/79.jpg)

@Deprecated是一种注释，表示不推荐使用。

![80.jpg](/images/blog_pic/JVM/类文件结构/80.jpg)

![81.jpg](/images/blog_pic/JVM/类文件结构/81.jpg)

![82.jpg](/images/blog_pic/JVM/类文件结构/82.jpg)

以下为其中常用的几种：

**Code**

Java方法的方法体会被以字节码指令的形式存储至class文件方法表的属性表集合中，其属性名为Code。

正如并非所有方法都有方法体一样(例如接口中的方法，或抽象类中的抽象方法)，并非所有方法表都必定包含Code属性。

Code属性的结构如下图所示：

![83.jpg](/images/blog_pic/JVM/类文件结构/83.jpg)

仍然使用常量池集合的代码为例，继续分析方法表集合中的方法1，我们已知其属性表集合中有1个属性表。现继续详细分析该属性表。

attribute_name_index及attribute_length为所有属性表的固定结构，前文已有描述。对于Code属性而言，attribute_name_index固定为Code：

![84.jpg](/images/blog_pic/JVM/类文件结构/84.jpg)

0x0009=9，即Code。

![85.jpg](/images/blog_pic/JVM/类文件结构/85.jpg)

0x0000001D=29。即该属性表的长度为29个字节，则我们可以确定出其外部范围：

![86.jpg](/images/blog_pic/JVM/类文件结构/86.jpg)

attribute_length之后的2个字节为max_stack，代表方法操作数栈(Operand Stacks)的最大深度，即操作数栈的大小。运行期该值即为方法的栈帧(Stack Frame)中的操作数栈的深度。示例代码中：

![87.jpg](/images/blog_pic/JVM/类文件结构/87.jpg)

0x0001=1，即方法操作数栈最大深度为1。

max_stack之后的2个字节为max_locals，表示局部变量表所需的存储空间。

局部变量表的基本单位为Slot。可以认为一个Slot为32位，因此对于byte,char,float,int,short,boolean,returnAddress这种长度不超过32位的数据类型而言，只需1个Slot即可。而对于double,long这两种64位的数据类型而言，占用2个Slot。

局部变量表用于存储如下信息：

- 方法参数(若为实例方法，还会包含默认隐藏的this)
- 显式异常处理器的参数(Exception Handler Parameter)，也就是catch括号中显式声明的异常
- 方法体中定义的局部变量

对于实例方法而言，初始时0号Slot中默认为this。

max_locals小于等于方法所用的数据项占用的空间之和，其原因在于每个数据项在方法中都有自身的作用域，若超出作用域，其所占据的空间就可以被其他数据项复用。

![88.jpg](/images/blog_pic/JVM/类文件结构/88.jpg)

0x0001=1，即局部变量表大小为1。

max_locals之后是字节码指令集合。其分为两部分：第一部分为4个字节的code_length，代表字节码指令流长度。第二部分为code_length*u1个字节，代表具体的字节码指令。

每个字节码指令的长度为u1。每当JVM读入一个字节码，都可以明白其所表征的含义，并能根据该字节码指令的定义了解其后是否需要跟随参数及这些参数的含义。

u1的无符号数的取值范围为[0,255]。即共可表示256种字节码指令。目前已使用了约200种。详见[JVM-JVM字节码指令集](/2017/11/16/JVM-JVM字节码指令集/)。

code_length的长度为u4，则理论上最大长度可达2^32-1。但是JVM规范中明确规定1个方法的字节码指令流最大长度为65535，即实际上code_length最大只使用了u2的空间，如果超出了这个限制，javac编译器也会拒绝编译。一般来讲，除非故意为难编译器，一个方法中的字节码指令流是不会超过这个上限的。但是，某些特殊情况，例如在编译一个很复杂的JSP文件时，某些JSP编译器会把JSP内容和页面输出的信息归并于一个方法之中，此时生成的方法中的字节码指令流的长度就有可能超过上限，从而编译失败。

Code属性是class文件中最重要的属性，而字节码指令集合又是Code属性中最为重要的部分。

我们可将1个Java程序分为两部分：

- 代码(Code)：方法体中的Java代码
- 元数据(Metadata)：类，字段，方法定义及其他信息

那么映射到class文件，Code属性用于描述方法体中的Java代码，而其他所有部分都是在描述元数据。

示例代码中：

![89.jpg](/images/blog_pic/JVM/类文件结构/89.jpg)

code_length=0x00000005=5，因此向后取5个字节作为字节码指令流：

![90.jpg](/images/blog_pic/JVM/类文件结构/90.jpg)

由[JVM-JVM字节码指令集](/2017/11/16/JVM-JVM字节码指令集/)得：

---

**2A(aload_0)**:将局部变量表0号Slot中的引用类型压入操作数栈，对于实例方法而言，初始时0号Slot中默认为this。

---

**B7(invokespecial)**:以操作数栈栈顶reference类型的数据所指向的对象为方法的接收者，调用此对象的实例构造器&lt;init&gt;方法，私有方法或超类构造方法。该指令之后会紧跟一个u2的参数说明具体调用的是哪个方法，该参数指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项，也就是该方法的方法符号引用。

0001:invokespecial的参数，即为：

```
java/lang/Object."<init>":()V
```

---

**B1(return)**:从当前方法返回void，本方法结束。

---

由上述分析可知，JVM执行字节码是基于栈的，但是与一般的基于栈的零字节指令(即每条指令只占一位，没有后续附加信息)所不同，某些指令(如invokespecial)后面还会带有参数。

字节码指令集合之后是异常表集合，其分为两部分，第一部分为2个字节的exception_table_length，代表异常表长度。第二部分为exception_table_length个exception_info。

异常表集合对于Code属性而言不是必须的，若没有该集合，则exception_table_length=0。继续分析示例代码：

![91.jpg](/images/blog_pic/JVM/类文件结构/91.jpg)

即exception_table_length=0。

exception_info的结构如下图所示：

![92.jpg](/images/blog_pic/JVM/类文件结构/92.jpg)

排列顺序为先左后右。其含义为：若[start_pc,end_pc)行之间的字节码出现了类型为catch_type(u2,指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项，若指向第0项，则表示能捕获任意异常)或其子类型的异常，则跳转至第handler_pc行。需要注意的是，此处字节码的行是一种形象化的表示，实际上是字节码较之方法体开始位置的偏移量。

class文件使用exception_info而非简单的跳转命令来实现Java的异常及finally机制。其实例可参见[Java 基础-异常](/2017/09/29/Java 基础-异常/)。

异常表集合之后是Code属性表内部包含的属性表集合。其内部分为两部分：首先是u2长度的attributes_count，随后是attributes_count个attribute_info类型。属性内部又有个属性表集合，听起来有点绕。但其套路与前文已讨论过的类文件，字段表，方法表一样，都是在结构的最后加一些不方便固定描述的额外信息。

继续分析示例代码第一个方法的Code属性：

![93.jpg](/images/blog_pic/JVM/类文件结构/93.jpg)

0x0001=1，即attributes_count=1，再向后取一个attribute_info类型，在该attribute_info内部，先取u2长度作为attribute_name_index：

![94.jpg](/images/blog_pic/JVM/类文件结构/94.jpg)

0x000A=10，即LineNumberTable。

attribute_name_index之后是u4长度的attribute_length：

![95.jpg](/images/blog_pic/JVM/类文件结构/95.jpg)

0x00000006=6，即attribute_length=6。正好蓝色的区域向后数6个也结束，说明Code属性也结束了，和预期相符。

---

**LineNumberTable**

LineNumberTable用于描述Java源码的行号与字节码行号(实际是相对于方法体开始处的字节码偏移量)之间的映射关系。

LineNumberTable并非是必须的，但默认会在class文件中生成，在使用javac命令进行编译时，可使用-g:none及-g:lines显式取消或要求生成该属性。若取消生成该属性，则当抛出异常时，堆栈中不会显示出错的行号，并且在调试程序时，也无法按照源码行设置断点。

LineNumberTable的结构如下：

![96.jpg](/images/blog_pic/JVM/类文件结构/96.jpg)

前两项u2长度的attribute_name_index及u4长度的attribute_length同其他属性一样，随后是行号表集合。

行号表集合内部由2部分组成，首先是u2长度的line_number_table_length，随后是line_number_table_length个line_number_info。

则继续分析示例代码中的Code属性：

![97.jpg](/images/blog_pic/JVM/类文件结构/97.jpg)

0x0001=1，即line_number_table_length=1。则向后取1个line_number_info。

在line_number_info内部，首先是u2长度的start_pc，代表字节码相对于方法体开始处的偏移量。随后是u2长度的line_number，代表Java源码行号。

则继续分析示例代码中的Code属性：

![98.jpg](/images/blog_pic/JVM/类文件结构/98.jpg)

0x0000=0，即start_pc=0。

![99.jpg](/images/blog_pic/JVM/类文件结构/99.jpg)

0x0003=3，即line_number=3。

至此，实例代码中的第一个方法所包含的Code属性已分析完毕，其结果与javap得到的信息相同：

```
Classfile /E:/Test.class
  Last modified 2017-11-8; size 274 bytes
  MD5 checksum ba2585a36b64eb15c6657dc8440c38e0
  Compiled from "Test.java"
public class com.test.Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         //  com/test/Test.m:I
   #3 = Class              #17            //  com/test/Test
   #4 = Class              #18            //  java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               Test.java
  #15 = NameAndType        #7:#8          //  "<init>":()V
  #16 = NameAndType        #5:#6          //  m:I
  #17 = Utf8               com/test/Test
  #18 = Utf8               java/lang/Object
{
  public com.test.Test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 3: 0

  public int inc();
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0       
         1: getfield      #2                  // Field m:I
         4: iconst_1      
         5: iadd          
         6: ireturn       
      LineNumberTable:
        line 8: 0
}
```

对应方法为public com.test.Test()，其中stack代表操作数栈深度，locals代表局部变量表大小，args_size代表传入参数大小。locals及args_size均为1的原因为该方法为实例方法，默认会传入其所属对象的引用this。此时this默认占用局部变量表0号索引的位置。

因示例代码方法表集合的长度为2，则我们继续向后分析第二个方法，第一个方法结束后，紧接着就是第二个方法的method_info。

在该method_info内部，首先是u2长度的access_flags：

![100.jpg](/images/blog_pic/JVM/类文件结构/100.jpg)

即access_flags=0x0001=0000_0000_0000_0001。则该方法的访问标志有ACC_PUBLIC。

access_flags之后是u2长度的name_index：

![101.jpg](/images/blog_pic/JVM/类文件结构/101.jpg)

0x000B=11，即inc。

name_index之后是u2长度的descriptor_index：

![102.jpg](/images/blog_pic/JVM/类文件结构/102.jpg)

0x000C=12，即()I。

至此我们已可得到第二个方法的定义信息：

```
public int inc()
```

随后就是方法2的属性表集合了。其内部首先是u2长度的attributes_count，代表其属性表集合的长度：

![103.jpg](/images/blog_pic/JVM/类文件结构/103.jpg)

attributes_count=0x0001=1。即该属性表内部只有一个attribute_info。则向后取一个attribute_info。

在该attribute_info内部，首先是u2长度的attribute_name_index:

![104.jpg](/images/blog_pic/JVM/类文件结构/104.jpg)

attribute_name_index=0x0009=9，即Code。随后是该attribute_info的u4长度的attribute_length：

![105.jpg](/images/blog_pic/JVM/类文件结构/105.jpg)

attribute_length=0x0000001F=31。即该Code属性的外部边界长度为31：

![106.jpg](/images/blog_pic/JVM/类文件结构/106.jpg)

接着我们详细分析该Code属性，首先是u2长度的max_stack：

![107.jpg](/images/blog_pic/JVM/类文件结构/107.jpg)

max_stack=0x0002=2，即操作数栈最大深度为2。

随后是u2长度的max_locals：

![108.jpg](/images/blog_pic/JVM/类文件结构/108.jpg)

max_locals=0x0001=1，即局部变量表最大占用空间为1个Slot。

随后是字节码指令集合。其内部首先是u4长度的code_length：

![109.jpg](/images/blog_pic/JVM/类文件结构/109.jpg)

code_length=0x00000007=7，即此后的7个字节为该方法方法体中代码的字节码指令流：

![110.jpg](/images/blog_pic/JVM/类文件结构/110.jpg)

依序分析这个字节码指令流，由[JVM-JVM字节码指令集](/2017/11/16/JVM-JVM字节码指令集/)得：

前文分析已得，该方法操作数栈最大深度为2，局部变量表最大占用空间为1个Slot。

---

初始时:

- 操作数栈：无
- 局部变量表：this

---

**2A(aload_0)**:将局部变量表0号Slot中的引用类型压入操作数栈:

- 操作数栈：this
- 局部变量表：this

---

**B4(getfield)**:获取指定类的实例域，并将其值压入操作数栈。该指令之后会紧跟一个u2的参数指明实例域的全限定名，该参数指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项。

0002:getfield的参数，即为：com/test/Test.m:I

执行该指令后：

- 操作数栈：m
- 局部变量表：this

---

**04(iconst_1)**:将int型1压入操作数栈:

- 操作数栈：m,1
- 局部变量表：this

---

**60(iadd)**:弹出操作数栈栈顶两int型数值相加并将结果压入操作数栈:

- 操作数栈：m+1
- 局部变量表：this

---

**AC(ireturn)**:返回操作数栈栈顶的int类型数值，即返回m+1

---

随后是异常表集合，其内部首先是u2长度的exception_table_length:

![111.jpg](/images/blog_pic/JVM/类文件结构/111.jpg)

exception_table_length=0x0000=0。即没有异常表，则异常表集合结束。

异常表集合之后是Code属性表内部包含的属性表集合。其内部分为两部分：首先是u2长度的attributes_count：

![112.jpg](/images/blog_pic/JVM/类文件结构/112.jpg)

0x0001=1，即attributes_count=1，再向后取一个attribute_info类型，在该attribute_info内部，先取u2长度作为attribute_name_index：

![113.jpg](/images/blog_pic/JVM/类文件结构/113.jpg)

0x000A=10，即LineNumberTable。

attribute_name_index之后是u4长度的attribute_length：

![114.jpg](/images/blog_pic/JVM/类文件结构/114.jpg)

0x00000006=6，即attribute_length=6。正好蓝色的区域向后数6个也结束，说明Code属性也结束了，和预期相符。

在LineNumberTable内部，首先是u2长度的line_number_table_length：

![115.jpg](/images/blog_pic/JVM/类文件结构/115.jpg)

line_number_table_length=0x0001=1。则向后取1个line_number_info。

在line_number_info内部，首先是u2长度的start_pc，代表字节码相对于方法体开始处的偏移量。随后是u2长度的line_number，代表Java源码行号。

则：

![116.jpg](/images/blog_pic/JVM/类文件结构/116.jpg)

start_pc=0x0000=0。

随后：

![117.jpg](/images/blog_pic/JVM/类文件结构/117.jpg)

line_number=0x0008=8。

至此，示例代码中的第二个方法所包含的Code属性已分析完毕，其结果与javap得到的信息相同，也就是其中的inc()：

```
Classfile /E:/Test.class
  Last modified 2017-11-8; size 274 bytes
  MD5 checksum ba2585a36b64eb15c6657dc8440c38e0
  Compiled from "Test.java"
public class com.test.Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         //  com/test/Test.m:I
   #3 = Class              #17            //  com/test/Test
   #4 = Class              #18            //  java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               Test.java
  #15 = NameAndType        #7:#8          //  "<init>":()V
  #16 = NameAndType        #5:#6          //  m:I
  #17 = Utf8               com/test/Test
  #18 = Utf8               java/lang/Object
{
  public com.test.Test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 3: 0

  public int inc();
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0       
         1: getfield      #2                  // Field m:I
         4: iconst_1      
         5: iadd          
         6: ireturn       
      LineNumberTable:
        line 8: 0
}

```

至此示例代码class文件的方法表集合彻底结束。随后就是class文件的最后一部分，即属于类的属性表集合。其内部分为两部分：首先是u2长度的attributes_count，随后是attributes_count个attribute_info类型。

则先取attributes_count：

![118.jpg](/images/blog_pic/JVM/类文件结构/118.jpg)

attributes_count=0x0001=1，即该属性表集合中只有一个attribute_info，则继续向后取该attribute_info。

在attribute_info内部，首先是u2长度的attribute_name_index：

![119.jpg](/images/blog_pic/JVM/类文件结构/119.jpg)

attribute_name_index=0x000D=13，即SourceFile。

随后是u4长度的attribute_length:

![120.jpg](/images/blog_pic/JVM/类文件结构/120.jpg)

attribute_length=0x00000002=2。

---

**SourceFile**

SourceFile用于记录生成该class文件的源文件的文件名称。本属性可选，但默认是开启的。在使用javac指令生成class文件时可通过-g:none或-g:source选项显式取消或要求生成该属性。若不生成该属性，当抛出异常时，堆栈中将不会显示出错代码所属的文件名。在Java中，多数情况下类名与其所属的文件名是一致的，但少数情况下(非public类或内部类)例外。

SourceFile为一个定长结构：

![121.jpg](/images/blog_pic/JVM/类文件结构/121.jpg)

sourcefile_index指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项。

继续分析示例代码：

![122.jpg](/images/blog_pic/JVM/类文件结构/122.jpg)

0x000E=14，即Test.java。

至此，示例代码全部分析完毕。

---

**Exceptions**

本属性与Code属性平级，同Code属性中的数据项异常表集合是不同的。

Exceptions的作用为列出方法中可能抛出的受查异常(Checked Exceptions)。更具体的来说，也就是列出方法定义中throws关键字之后跟随的受查异常。其结构为：

![123.jpg](/images/blog_pic/JVM/类文件结构/123.jpg)

排列顺序为先左后右。该属性的前两项attribute_name_index及attribute_length为所有属性的通用数据项。随后跟随的为受查异常集合。

在受查异常集合内部，首先为u2长度的number_of_exceptions，即受查异常个数。随后是number_of_exceptions个u2长度的exception_index_table，每个exception_index_table都指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项，代表每个受查异常。

---

**LocalVariableTable**

LocalVariableTable用于描述栈帧局部变量表中的变量与Java源码中定义的变量之间的关系。其不是必须的属性，默认不会生成在class文件中。在用javac命令生成class文件时可通过-g:none及-g:vars显式取消或生成本属性。

若没有生成该属性，当其他人引用该方法时，所有参数的名称都将丢失，IDE将会使用诸如arg0,arg1之类的占位符代替原有的参数名。取消本属性不会影响程序的运行，但显然会对编码造成较大不便，且在调试期间也无法根据参数名从上下文中获取参数值。

LocalVariableTable的结构如下图所示：

![124.jpg](/images/blog_pic/JVM/类文件结构/124.jpg)

该属性的前两项attribute_name_index及attribute_length为所有属性的通用数据项。随后跟随的为本地变量表集合。

在本地变量表集合内部，首先为u2长度的local_variable_table_length，随后为local_variable_table_length个local_variable_table。local_variable_table代表一个源码局部变量到栈帧中的局部变量的映射关系，其结构如下图所示：

![125.jpg](/images/blog_pic/JVM/类文件结构/125.jpg)

start_pc代表该局部变量生命周期开始时相对于方法体的字节码偏移量。length代表该局部变量的作用范围长度。二者结合即可确定该局部变量在方法体中的作用域。

name_index及descriptor_index均指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项。分别表示该局部变量的名称及描述符。

index是该局部变量在栈帧局部变量表中的Slot位置。当该局部变量为64位(long或double)时，将占用两个Slot。此时index中记录的是第一个Slot索引值。

顺带一提，自JDK1.5引入泛型后，便增加了一个LocalVariableTable的姊妹属性：LocalVariableTypeTable。其与LocalVariableTable非常相似，不同之处仅仅在于其将局部变量的描述符descriptor_index替换为了局部变量的特征签名(Signature)。对于非泛型类型而言，描述符和特征签名能描述的信息是基本一致的。但是引入泛型后，由于描述符中泛型的参数化类型被擦除了，描述符就无法准确的描述泛型类型了。因此才出现了LocalVariableTypeTable。

---

**ConstantValue**

根据JVM规范，ConstantValue属性的作用是通知JVM自动为静态变量赋值。只有被static修饰的变量(即类变量)才能使用该属性。

若变量定义如下：

```
int x = 123;
```

这被称为实例变量，其是在实例构造器&lt;init&gt;中赋值的。

而对于类变量(即上文x再被static修饰)，赋值时机有两个：类构造器&lt;clinit&gt;或使用ConstantValue属性。对于SUN自带的javac编译器而言，对于类变量，如果同时满足如下两个条件：

- 使用final修饰
- 数据类型为基本类型或java.lang.String

则使用ConstantValue属性初始化，反之则在&lt;clinit&gt;中初始化。

再次强调，JVM规范与JVM实现是不同的。对于ConstantValue属性，JVM规范只设置static一个限制条件，至于final及数据类型则是SUN实现的javac编译器自身追加的限制。

ConstantValue属性的结构如下图所示：

![126.jpg](/images/blog_pic/JVM/类文件结构/126.jpg)

很显然，这是一个定长属性。constantvalue_index表示常量值，其指向常量池集合中的一个索引项。这也是为什么ConstantValue属性会有类型限制的原因：因为常量池集合中只能存储基本数据类型及字符串类型，即便ConstantValue属性想表示其他类型也无从下手。具体来说，常量池集合中能表示的类型有如下5种：

- CONSTANT_Double_info
- CONSTANT_Float_info
- CONSTANT_Long_info
- CONSTANT_Integer_info
- CONSTANT_String_info

---

**InnerClasses**

InnerClasses属性用于记录内部类与宿主类之间的关联。如果一个类包含了内部类，那么编译器会为它及它所包含的内部类生成本属性。InnerClasses属性的结构如下图所示：

![127.jpg](/images/blog_pic/JVM/类文件结构/127.jpg)

在所有属性共有的头两个数据项之后是内部类集合。其由两部分组成，第1部分是u2长度的number_of_classes，代表需要记录多少个内部类信息，随后是number_of_classes个inner_classes_info。inner_classes_info的结构如下图所示：

![128.jpg](/images/blog_pic/JVM/类文件结构/128.jpg)

inner_class_info_index及outer_class_info_index均指向常量池集合中的一个CONSTANT_Class_info类型的索引项，分别代表内部类及宿主类的符号引用。

inner_name_index指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项，代表内部类的名称。若其为匿名内部类，则指向常量池集合的索引0。

inner_class_access_flags是内部类的访问标志，类似于类的access_flags，其取值范围如下图所示：

![129.jpg](/images/blog_pic/JVM/类文件结构/129.jpg)

---

**Deprecated及Synthetic**

Deprecated及Synthetic均是标志类型的布尔属性，只有是否存在的区别，没有具体的属性值：有该属性就被认为是true，没有该属性就是false。

Deprecated用于表示被其修饰的类，字段或方法已被代码作者定为不推荐使用，体现在代码中，就是``` @deprecated ```注解。

Synthetic表示字段或方法不是依据Java源码生成的，而是由编译器自行添加的。

自JDK1.5起，标志一个类，字段或方法是由编译器自动生成的，也可在它们的访问标志access_flags中设置ACC_SYNTHETIC标志位，其中最典型的例子就是Bridge Method(桥接方法是JDK1.5引入泛型后，为了使Java的泛型方法生成的字节码和JDK1.5前的字节码相兼容，由编译器自动生成的方法)。

除了类构造器方法&lt;clinit&gt;及实例构造器方法&lt;init&gt;之外，其他所有由非源码产生的类，方法及字段都应至少设置Synthetic属性或ACC_SYNTHETIC标志位中的一项。

Deprecated及Synthetic的结构均如下图所示：

![130.jpg](/images/blog_pic/JVM/类文件结构/130.jpg)

即只有所有属性共有的那两个数据结构，且必有attribute_length=0。

---

**StackMapTable**

自JDK1.6起StackMapTable被加入到class文件规范中，并于JDK1.7时强制代替了原本的基于类型推断的字节码验证器。它是一个复杂的变长属性，位于Code属性的属性表集合中。

StackMapTable属性会在运行期类加载中的连接中的字节码验证阶段被新类型检查验证器(Type Checker)使用。目的在于代替以前比较消耗性能的基于数据流分析的类型推导验证器。简单来说，就是将验证所需的信息的生成及验证逻辑尽可能的提前到编译阶段并形成StackMapTable属性。

StackMapTable属性的结构如下图所示：

![131.jpg](/images/blog_pic/JVM/类文件结构/131.jpg)

在所有属性共有的头两个数据项之后是栈映射帧(Stack Map  Frames)集合。其内部由两部分组成：首先是u2长度的number_of_entries，而后是number_of_entries个stack_map_frame。stack_map_frame也就是所谓的栈映射帧。每个栈映射帧都显式或隐式的包含了一个字节码偏移量，用于表示执行到该字节码时局部变量表及操作数栈需满足的验证类型。

《Java虚拟机规范(Java SE 7版)》中规定，版本号大于等于50(即JDK1.6)的class文件，如果方法Code属性中没有包含StackMapTable属性，那么意味着其包含一个隐式的StackMapTable属性。该StackMapTable属性的number_of_entries=0。该规范同样也规定一个方法的Code属性的属性集合中最多只能包含一个StackMapTable属性，否则将抛出ClassFormatError。

---

**Signature**

Signature属性是JDK1.5新增的属性，这是一个可选的定长属性，可能出现于类，字段表及方法表的属性表集合中。

JDK1.5中引入了一个重要的特性-泛型。但是Java语言中的泛型是采用擦除法实现的伪泛型，在Code属性中，泛型信息(类型变量[Type Variables]，参数化类型[Parameterized Types])在编译后都会被通通擦除掉。使用擦除法实现泛型的好处在于简单，较之此前没有泛型的版本修改较小(主要只需修改javac编译器，JVM内部改动很小)，同时也非常易于实现Backport(Backport是将一个软件的补丁应用到比此补丁所对应的版本更老的版本的行为)，运行期也能节省一些类型所占用的内存空间(这个好处听起来就很牵强了，一般来说不会在乎那么点内存的)。然而其坏处却是致命的，也是Java一直被人诟病的点之一：假的就是假的，用一个伪物试图实现真物的功能必然要付出很大的代价，而且即便付出代价了，其结果往往也是似是而非。Java无法像C#等支持真泛型的语言那样，在运行期将泛型类型与用户定义的普通类型同等对待，起码无法走正常渠道直接在运行期通过反射获得泛型信息。

Signature属性就是这个异常渠道，或者说是试图用伪物替代真物所付出的代价。任何类，接口，方法或字段如果包含了泛型信息都需记录到Signature属性中。换句话说，该存类型信息的地方因为不想做过多的修改把泛型信息擦除了，而使用的地方又想用，那么怎么办呢？只能再找个地方把泛型信息存起来。Java的反射API能获取泛型类型，最终的数据来源也就是这个属性。

Signature属性的结构如下图所示：

![132.jpg](/images/blog_pic/JVM/类文件结构/132.jpg)

在所有属性共有的头两个数据项之后是u2长度的signature_index。其指向常量池集合中的一个CONSTANT_Utf8_info类型的索引项，若该Signature属性是类文件的属性，则其表示类签名。若该Signature属性是方法表的属性，则其表示方法类型签名。若该Signature属性是字段表的属性，则其表示字段类型签名。

---

**BootstrapMethods**

BootstrapMethods属性是JDK1.7新增的属性，它是一个复杂的变长属性，位于类文件的属性表集合中。该属性用于存储invokedynamic指令引用的引导方法限定符。《Java虚拟机规范(Java SE 7版)》规定，如果常量池集合中出现过CONSTANT_InvokeDynamic_info类型的常量，那么类文件的属性表集合中必须出现且只能出现一个BootstrapMethods属性。

BootstrapMethods属性与invokedynamic指令及java.lang.invoke包的关系非常密切。截至JDK1.7为止，javac编译器尚无法生成invokedynamic指令及BootstrapMethods属性，必须通过一些非常规的手段才能使用到它们。

BootstrapMethods属性的结构如下图所示：

![133.jpg](/images/blog_pic/JVM/类文件结构/133.jpg)

在所有属性共有的头两个数据项之后是引导方法集合。其内部由两部分组成：首先是u2长度的num_bootstrap_methods，随后是num_bootstrap_methods个bootstrap_method。bootstrap_method的结构如下图所示：

![134.jpg](/images/blog_pic/JVM/类文件结构/134.jpg)

首先是u2长度的bootstrap_method_ref，其指向常量池集合中的一个CONSTANT_MethodHandle_info类型的索引项。随后是引导参数集合。在引导参数集合内部，首先是u2长度num_bootstrap_arguments，其值可以为0，表示没有引导参数。随后是num_bootstrap_arguments个u2长度的bootstrap_argument。每个bootstrap_argument都指向常量池集合中的一个数据项，且必须为如下常量中的一种：

- CONSTANT_String_info
- CONSTANT_Class_info
- CONSTANT_Integer_info
- CONSTANT_Long_info
- CONSTANT_Float_info
- CONSTANT_Double_info
- CONSTANT_MethodHandle_info
- CONSTANT_MethodType_info