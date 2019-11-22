---
title: Java JDK7源码-java.io.Serializable
date: 2017-07-04 18:31:36
tags: [Java,源码]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.io;

public interface Serializable {
}
```

# 已整理层级关系

***直接实现本接口的类***

- [java.util.ArrayList&lt;E&gt;](/2017/07/06/Java JDK7源码-javautilArrayListE/)
- [java.util.HashMap&lt;K,V&gt;](/2018/11/09/Java JDK7源码-javautilHashMapKV/)

# 综述

实现本接口的类具备可序列化的能力。未实现本接口的类则不能进行序列化及反序列化。所有可序列化的类的子类都自动具备序列化的能力而无需在类定义中声明。本接口中没有任何方法或字段，它只是一个标志，标识实现本接口的类具备可序列化的能力。

为使非序列化的类具备序列化的能力，通常的做法为声明一个具备序列化能力的子类，子类必须承担起保存及恢复子类的public，protected，以及(如果可访问的话)封装字段的状态的责任。子类只在如下条件下才能承担起该责任：该子类的超类中有一个可访问的无参构造函数用以帮助子类初始化自身的状态，若不满足该条件则可通过编译，但在运行时会抛出异常。

在反序列化的过程中，非序列化类的字段将被该类的public或protected的无参构造函数初始化。该无参构造函数必须要能够被已实现序列化的子类访问。已实现序列化的子类的字段将从流中恢复。

在传输图结构时，可能会遇到不支持本接口的情况。此时会抛出NotSerializableException并定位到未序列化的类。

# writeObject,readObject,readObjectNoData

在序列化及反序列化时需要进行特殊操作的类必须实现如下特定的方法：

```
private void writeObject(java.io.ObjectOutputStream out)
    throws IOException
private void readObject(java.io.ObjectInputStream in)
    throws IOException, ClassNotFoundException;
private void readObjectNoData()
    throws ObjectStreamException;
```

writeObject负责写特定的类的对象的状态以便于对应的readObject可以恢复它。默认情况下是通过调用out.defaultWriteObject()保存对象的状态。本方法不需要关心其所属超类或子类的状态。状态会以独立字段的形式被本方法或out.defaultWriteObject()存入ObjectOutputStream。默认写入的内容为调用对象的toString()的返回值。测试如下：

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class STest {

    private static final String PATH_F = "D://data.f";

    private static void writeByFOS(Data data, String path) throws Exception {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(path));
        objectOutputStream.writeObject(data);
        objectOutputStream.close();
    }

    private static Data readByFIS(String path) throws Exception {
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(path));
        Data result = (Data)objectInputStream.readObject();
        objectInputStream.close();
        return result;
    }

    public static void main(String args[]) throws Exception {
        STest.writeByFOS(new Data(0, "灵梦"), STest.PATH_F);
        System.out.println(STest.readByFIS(STest.PATH_F));
    }
}

class Data implements Serializable {

    private static final long serialVersionUID = 615562960517507579L;

    private int id;

    private String value;

    public Data(int id, String value) {
        this.id = id;
        this.value = value;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Data [id=" + (id + 1) + ", value=" + value + value + "]";
    }
}
```

输出结果：

```
Data [id=1, value=灵梦灵梦]
```

readObject方法负责从流中读取数据并恢复类字段。本方法可以调用in.defaultReadObject()以采用默认的机制恢复对象的non-static及non-transient字段。defaultReadObject()将流中存储的对象的字段存入当前对象的对应位置。本方法处理类需要加字段的情况。本方法不需要关心其所属超类或子类的状态。状态会以独立字段的形式被writeObject方法或out.defaultWriteObject()存入ObjectOutputStream。

writeObject及readObject示例如下： 

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class STest {

    private static final String PATH_F = "D://data.f";

    private static void writeByFOS(Data data, String path) throws Exception {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(path));
        objectOutputStream.writeObject(data);
        objectOutputStream.close();
    }

    private static Data readByFIS(String path) throws Exception {
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(path));
        Data result = (Data)objectInputStream.readObject();
        objectInputStream.close();
        return result;
    }

    public static void main(String args[]) throws Exception {
        STest.writeByFOS(new Data(0, "灵梦"), STest.PATH_F);
        System.out.println(STest.readByFIS(STest.PATH_F));
    }
}

class Data implements Serializable {

    private static final long serialVersionUID = 615562960517507579L;

    private int id;

    private String value;

    public Data(int id, String value) {
        this.id = id;
        this.value = value;
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Data [id=" + id + ", value=" + value + "]";
    }
}
```

此时内部均仍采用默认机制。结果如下：

```
Data [id=0, value=灵梦]

```

若将writeObject方法体置为空，则抛出异常：

```
Exception in thread "main" java.io.EOFException
	at java.io.ObjectInputStream$BlockDataInputStream.readFully(ObjectInputStream.java:2744)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:1979)
	at java.io.ObjectInputStream.defaultReadObject(ObjectInputStream.java:500)
	at stest.Data.readObject(STest.java:51)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at java.io.ObjectStreamClass.invokeReadObject(ObjectStreamClass.java:1017)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1893)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1798)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:370)
	at stest.STest.readByFIS(STest.java:22)
	at stest.STest.main(STest.java:29)
```

若将readObject置为空，则抛出异常：

```
Exception in thread "main" java.io.StreamCorruptedException: invalid type code: 00
	at java.io.ObjectInputStream$BlockDataInputStream.readBlockHeader(ObjectInputStream.java:2508)
	at java.io.ObjectInputStream$BlockDataInputStream.refill(ObjectInputStream.java:2543)
	at java.io.ObjectInputStream$BlockDataInputStream.skipBlockData(ObjectInputStream.java:2445)
	at java.io.ObjectInputStream.skipCustomData(ObjectInputStream.java:1941)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:1918)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:1798)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1350)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:370)
	at stest.STest.readByFIS(STest.java:22)
	at stest.STest.main(STest.java:29)
```

Serializable对象反序列化时，由于序列化与反序列化提供的class版本不同，序列化的class的super class不同于反序列化时的class的super class；或待接收的流受到了干扰；或者收到有敌意的流；或接收不完整；都会对初始化对象字段值时造成影响。如果发生以上情况时，没有定义readObjectNoData方法时，类的字段就会初始化成它们的默认值。当出现上面的情况时，readObjectNoData会取代readObject的调用。

# writeReplace,readResolve

当实现序列化的类需指定一个替代类时，在写入流时需实现以下方法：

```
ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;
```

若序列化的类未实现writeReplace方法则会调用writeObject方法写入流。若序列化的类实现了writeReplace方法则会用writeReplace方法替代writeObject方法写入流，测试用例如下：

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.ObjectStreamException;
import java.io.Serializable;

public class STest {

    private static final String PATH_F = "D://data.f";

    private static void writeByFOS(Data data, String path) throws Exception {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(path));
        objectOutputStream.writeObject(data);
        objectOutputStream.close();
    }

    private static Data readByFIS(String path) throws Exception {
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(path));
        Data result = (Data)objectInputStream.readObject();
        objectInputStream.close();
        return result;
    }

    public static void main(String args[]) throws Exception {
        STest.writeByFOS(new Data(0, "灵梦"), STest.PATH_F);
        System.out.println(STest.readByFIS(STest.PATH_F));
    }
}

class Data implements Serializable {

    private static final long serialVersionUID = 615562960517507579L;

    private int id;

    private String value;

    public Data(int id, String value) {
        this.id = id;
        this.value = value;
    }

    public Object writeReplace() throws ObjectStreamException {
        return new Data(1, this.value + this.value);
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Data [id=" + id + ", value=" + value + "]";
    }
}
```

输出结果为：

```
Data [id=1, value=灵梦灵梦]
```

```
ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;
```

同理，当反序列化时，要将一个对象从流中读出来，我们如果想将读出来的对象用另一个对象实例替换，则需实现readResolve方法。测试用例如下：

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.ObjectStreamException;
import java.io.Serializable;

public class STest {

    private static final String PATH_F = "D://data.f";

    private static void writeByFOS(Data data, String path) throws Exception {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(path));
        objectOutputStream.writeObject(data);
        objectOutputStream.close();
    }

    private static Data readByFIS(String path) throws Exception {
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(path));
        Data result = (Data)objectInputStream.readObject();
        objectInputStream.close();
        return result;
    }

    public static void main(String args[]) throws Exception {
        STest.writeByFOS(new Data(0, "灵梦"), STest.PATH_F);
        System.out.println(STest.readByFIS(STest.PATH_F));
    }
}

class Data implements Serializable {

    private static final long serialVersionUID = 615562960517507579L;

    private int id;

    private String value;

    public Data(int id, String value) {
        this.id = id;
        this.value = value;
    }

    public Object readResolve() throws ObjectStreamException {
        return new Data(1, this.value + this.value);
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Data [id=" + id + ", value=" + value + "]";
    }
}
```

输出结果为：

```
Data [id=1, value=灵梦灵梦]
```

# serialVersionUID

序列化运行时会将每一个实现序列化的类与一个版本号关联起来，称为serialVersionUID。该版本号被用于比对发送方及接受方所用的类是否是同一个版本。若接收方与发送方版本号不一致，则会抛出InvalidClassException。

实现序列化的类可指定serialVersionUID的值：

```
ANY-ACCESS-MODIFIER static final long serialVersionUID = xxxL;
```

若实现序列化的类未明确指定serialVersionUID，则序列化会在运行时依据待序列化类的各个方面计算一个默认的serialVersionUID值。然而Java强烈推荐每一个可序列化的类都明确指定自己的serialVersionUID，因为默认serialVersionUID的计算结果对类的编译结果异常敏感，因此若发送方及接收方的编译器有所差异则可能会在反序列化时抛出InvalidClassException。因此，为保证有一个无关Java编译实现的常量serialVersionUID值，序列化的类必须明确指定serialVersionUID。同时也强烈建议如果可能的话将serialVersionUID的访问权限设为private，因为serialVersionUID仅仅对其所属的类本身有用，换句话说，serialVersionUID并不是一个需继承的字段。数组类无法声明一个特定的serialVersionUID，所以它们总是使用默认的计算值，但是数组类并不需要与serialVersionUID建立起匹配关系。

# 测试

**代码**

```
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

public class STest {

    private static final String PATH_F = "D://data.f";

    private static void writeByFOS(List<Data> list, String path) throws Exception {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(path));
        objectOutputStream.writeObject("使用FOS:");
        objectOutputStream.writeObject(list);
        objectOutputStream.close();
    }

    @SuppressWarnings("unchecked")
    private static List<Data> readByFIS(String path) throws Exception {
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(path));
        System.out.println((String)objectInputStream.readObject());
        List<Data> result = (ArrayList<Data>)objectInputStream.readObject();
        objectInputStream.close();
        return result;
    }

    private static ByteArrayOutputStream writeByBAOS(List<Data> list) throws Exception {
        ByteArrayOutputStream result = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(result);
        objectOutputStream.writeObject("使用BAOS:");
        objectOutputStream.writeObject(list);
        objectOutputStream.flush();
        return result;
    }

    @SuppressWarnings("unchecked")
    private static List<Data> readByBAIS(ByteArrayOutputStream baos) throws Exception {
        ObjectInputStream objectInputStream = new ObjectInputStream(new ByteArrayInputStream(baos.toByteArray()));
        System.out.println((String)objectInputStream.readObject());
        List<Data> result = (ArrayList<Data>)objectInputStream.readObject();
        objectInputStream.close();
        return result;
    }

    public static void main(String args[]) throws Exception {
        List<Data> list = new ArrayList<Data>();
        list.add(new Data(0, "灵梦"));
        list.add(new Data(1, "v2"));
        list.add(new Data(2, "v3"));
        list.add(new Data(3, "v4"));

        STest.writeByFOS(list, STest.PATH_F);
        System.out.println(STest.readByFIS(STest.PATH_F));

        ByteArrayOutputStream baos = STest.writeByBAOS(list);
        System.out.println(STest.readByBAIS(baos));
    }
}

class Data implements Serializable {

    private static final long serialVersionUID = 615562960517507579L;

    private int id;

    private String value;

    public Data(int id, String value) {
        this.id = id;
        this.value = value;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Data [id=" + id + ", value=" + value + "]";
    }
}
```

**测试结果**

```
使用FOS:
[Data [id=0, value=灵梦], Data [id=1, value=v2], Data [id=2, value=v3], Data [id=3, value=v4]]
使用BAOS:
[Data [id=0, value=灵梦], Data [id=1, value=v2], Data [id=2, value=v3], Data [id=3, value=v4]]
```

**备注**

若Data类不实现Serializable接口，则可通过编译，但运行时会报如下异常：

```
Exception in thread "main" java.io.NotSerializableException: stest.Data
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1183)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:347)
	at java.util.ArrayList.writeObject(ArrayList.java:742)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at java.io.ObjectStreamClass.invokeWriteObject(ObjectStreamClass.java:988)
	at java.io.ObjectOutputStream.writeSerialData(ObjectOutputStream.java:1495)
	at java.io.ObjectOutputStream.writeOrdinaryObject(ObjectOutputStream.java:1431)
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1177)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:347)
	at stest.STest.writeByFOS(STest.java:19)
	at stest.STest.main(STest.java:57)
```