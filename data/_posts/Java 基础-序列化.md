---
title: Java 基础-序列化
date: 2017-09-28 15:32:49
tags: [java,序列化]
categories: Java 基础
---

# 如何实现对象克隆？

<!-- more -->

有两种方式：

- 实现Cloneable接口并重写Object类中的clone()。Object类中的clone()是浅克隆。

- 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆，代码如下。

```
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
 
public class CloneBySerializable {

    @SuppressWarnings("unchecked")
    public static <T> T clone(T object) {
        if (null == object) return null;
        try {
            // 调用ByteArrayInputStream或ByteArrayOutputStream对象的close方法没有任何意义
            // 这两个基于内存的流只要垃圾回收器清理对象就能够释放资源，这一点不同于对外部资源(如文件流)的释放
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            new ObjectOutputStream(byteArrayOutputStream).writeObject(object);
     
            ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
            return (T) new ObjectInputStream(byteArrayInputStream).readObject();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    public static void main(String[] args) {
        Person person = new Person("八云紫", new Book("永远的17岁"));
        System.out.println("原始：" + person);
        Person personCopy = CloneBySerializable.clone(person);
        System.out.println("copy：" + personCopy);
        personCopy.name = "八云紫（笑）";
        personCopy.book.name = "其实是个老婆婆";
        System.out.println("copy改变后-原始：" + person);
        System.out.println("copy改变后-copy：" + personCopy);
    }
}

class Person implements Serializable {

    private static final long serialVersionUID = -9198259586220345266L;

    public String name;

    public Book book;

    public Person(String name, Book book) {
        this.name = name;
        this.book = book;
    }

    @Override
    public String toString() {
        return "Person [name=" + name + ", book=" + book + "]";
    }
}


class Book implements Serializable {

    private static final long serialVersionUID = -8536837238716978714L;

    public String name;

    public Book(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Book [name=" + name + "]";
    }
}
```

输出：

```
原始：Person [name=八云紫, book=Book [name=永远的17岁]]
copy：Person [name=八云紫, book=Book [name=永远的17岁]]
copy改变后-原始：Person [name=八云紫, book=Book [name=永远的17岁]]
copy改变后-copy：Person [name=八云紫（笑）, book=Book [name=其实是个老婆婆]]
```

基于序列化和反序列化实现的克隆不仅仅是深度克隆，更重要的是通过泛型限定，可以检查出要克隆的对象是否支持序列化，这项检查是编译器完成的，不是在运行时抛出异常，这种方案明显优于使用Object类的clone方法克隆对象。让问题在编译的时候暴露出来总是优于把问题留到运行时。

# Java中如何实现序列化，有什么意义？

序列化就是一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化。可以对流化后的对象进行读写操作，也可将流化后的对象传输于网络之间。序列化是为了解决对象流读写操作时可能引发的问题(如果不进行序列化可能会存在数据乱序的问题)。

要实现序列化，需要让一个类实现Serializable接口，该接口是一个标识性接口，标注该类对象是可被序列化的，然后使用一个输出流来构造一个对象输出流并通过writeObject(Object)方法就可以将实现对象写出(即保存其状态)；如果需要反序列化则可以用一个输入流建立对象输入流，然后通过readObject方法从流中读取对象。