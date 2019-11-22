---
title: Java 设计模式-1.Iterator模式
date: 2018-06-05 17:37:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Iterator模式被归入了第1部分[适应设计模式]()。在GoF原书中，Iterator模式则被归入了[行为型设计模式]()。简单来说，Iterator模式可以被描述为：从含有多个元素的集合中将各个元素逐一取出来。也就是所谓的"一个一个遍历"。

<!-- more -->

# 综述

迭代输出数组中的元素是很常见的需求，在Java中，我们通常会这样做：

```
for (int i = 0; i < arr.length; i++)
    System.out.println(arr[i]);
```

如果我们将循环变量i的作用抽象化通用化，再将数组扩展为任意其他的集合性质的容器，就可以形成一种模式，GoF书将其称为Iterator模式。该模式的目的就是按照要求的顺序在集合中遍历元素。

iterate这个单词的含义是"反复做某件事"。在计算机领域，它的含义是"迭代"。自然而言的，iterator被称作"迭代器"。

# 示例程序

本示例程序会模拟一个书架(BookShelf)，书本(Book)会被放置到书架上。

首先，我们来总览一下本程序的类图：

![0.jpg](/images/blog_pic/Java 设计模式/1Iterator模式/0.jpg)

本程序中的所有代码将被统一置于design1包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/1Iterator模式/1.jpg)

其中，Main.java是用来测试的类，并未出现在类图中。

BookShelfIterator作为非public类存在于BookShelf.java文件中。

下面将逐个贴出每个类的源码。

**Aggregate接口**

```
package design1;

/**
 * Aggregate有"使聚合，集合"的意思
 * 顾名思义，实现了该接口的类将成为一个可以保存多个元素的集合
 */
public interface Aggregate<E> {

    /**
     * 生成并返回一个用于遍历集合的迭代器
     * @return Iterator<E>
     */
    Iterator<E> iterator();
}
```

**Iterator接口**

Iterator接口用于遍历集合中的元素，其作用相当于本文开篇中介绍的for循环中的那个i：

```
package design1;

public interface Iterator<T> {

    /**
     * 若iterator游标当前所指向的位置还有元素则返回true，反之返回false
     * 若在本方法返回false时继续调用next()方法，则next()方法会抛出异常
     * @return boolean, true - 存在下一个元素
     *                  false - 不存在下一个元素
     */
    boolean hasNext();

    /**
     * 返回iterator游标当前所指向的元素，随后游标后移一位
     * 
     * @throws NoSuchElementException iterator游标当前所指向的位置已无元素
     */
    T next();
}
```

**Book类**

```
package design1;

public class Book {

    private String name;

    public Book(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

**BookShelf类**

```
package design1;

import java.util.NoSuchElementException;

public class BookShelf<E> implements Aggregate<E> {

    private Book[] books;

    /**
     * int, 书架上已有书数
     */
    private int last;

    public BookShelf(int maxsize) {
        if (maxsize < 0)
            throw new IllegalArgumentException("bookShelf 's maxsize < 0");
        this.books = new Book[maxsize];
    }

    public void appendBook(Book book) {
        if (null == book)
            throw new NullPointerException("book is null");
        if (this.last == this.books.length)
            throw new IllegalStateException("bookShelf is full");
        this.books[last++] = book;
    }

    @Override
    public Iterator<E> iterator() {
        return new BookShelfIterator<E>(this);
    }

    int getLength() {
        return this.last;
    }

    Book getBookAt(int index) {
        if (index < 0 || index >= books.length)
            throw new IllegalArgumentException("index < 0 || index >= bookShelf 's length");
        return this.books[index];
    }
}
```

**BookShelfIterator类**

BookShelfIterator作为非public类存在于BookShelf.java文件中：

```
class BookShelfIterator<E> implements Iterator<E> {

    private BookShelf<E> bookShelf;

    /**
     * int, 迭代器当前游标指向的位置，即下一次调用next()方法时返回的元素的索引
     */
    private int index;

    BookShelfIterator(BookShelf<E> bookShelf) {
        this.bookShelf = bookShelf;
    }

    @Override
    public boolean hasNext() {
        return this.bookShelf.getLength() > this.index;
    }

    @SuppressWarnings("unchecked")
    @Override
    public E next() {
        if (!this.hasNext())
            throw new NoSuchElementException("no more element for bookShelf");
        return (E)this.bookShelf.getBookAt(index++);
    }
}
```

**Main类**

```
package design1;

public class Main {

    public static void main(String[] args) {
        BookShelf<Book> bookShelf = new BookShelf<Book>(5);
        bookShelf.appendBook(new Book("秀逗魔导士"));
        bookShelf.appendBook(new Book("9S"));
        bookShelf.appendBook(new Book("为美好的世界献上祝福"));
        bookShelf.appendBook(new Book("犬神"));
        bookShelf.appendBook(new Book("钢铁白兔骑士团"));
        Iterator<Book> iterator = bookShelf.iterator();
        while (iterator.hasNext())
            System.out.println(iterator.next().getName());
    }
}
```

执行Main.java，输出：

```
秀逗魔导士
9S
为美好的世界献上祝福
犬神
钢铁白兔骑士团
```

# 登场角色

上面的示例程序介绍了Iterator模式的Java实现，下面咱们试着跳出语言层面，抽象出Iterator模式中登场的角色。

**Iterator(迭代器)**

该角色负责制定按特定顺序逐个遍历集合中的元素的功能的约束，不针对特定的集合，也不提供具体的实现。在示例程序中，由Iterator接口扮演这个角色，并提供了出演这个角色须达到的最低标准：即对外提供hasNext()与next()这两个方法。其中hasNext()方法负责判断是否仍有下一个元素(即控制迭代何时结束)，next()负责切实的取到当前迭代出的元素。

**ConcreteIterator(具体的迭代器)**

该角色负责针对特定的集合，提供定制化的Iterator的实现。在示例程序中，由BookShelfIterator类扮演这个角色。它是专门为了迭代BookShelf这个集合而定制的Iterator，因为它只用于迭代BookShelf，因此也可被称为"BookShelf的专属迭代器"，可以被视为BookShelf的附属。因此在编写代码时，并没有为BookShelfIterator类专门创建一个文件，而是将其作为非public类放到了BookShelf.java中。

一个BookShelf实例可以创建多个BookShelfIterator实例，而对于每个BookShelfIterator实例而言，终其一生将只为一个BookShelf实例服务。因此在BookShelfIterator类的构造函数中，我们会传入BookShelf实例。并且当BookShelf类的代码发生变化时，BookShelfIterator也要做出相应的调整。

BookShelfIterator在实现了Iterator后，并未增加新的方法。也就是说只提供向后迭代(具体来说，是沿着数组索引增大的方向迭代)。事实上，对于特定的迭代器实现而言，可以根据具体的待迭代的集合的实现，设计不同的迭代方式。例如我们还可以提供向前迭代方法previous()，此时迭代将顺着索引减小的方式进行，相应的我们还要提供判断向前迭代结束条件的方法hasPrevious()。

**Aggregate(集合)**

该角色负责制定作为一个集合该有的基本约束，并不提供具体的实现。换句话说，某个具体的类只要实现了该角色，我们就可以将该类称为集合了。在示例程序中，由Aggregate接口扮演这个角色，并提供了出演这个角色须达到的最低标准：即提供iterator()方法。该方法返回一个从属于本集合的迭代器，我们可以通过它对集合完成迭代。

**ConcreteAggregate(具体的集合)**

该角色负责根据Aggregate这个角色的约束条件，结合具体的需求，生成一个特定的集合类。因为它满足了Aggregate这个角色的约束，因此它会实现iterator()方法，该方法返回的迭代器自然就是ConcreteIterator。在示例程序中，由BookShelf类负责扮演这个角色，它的iterator()方法返回的迭代器是BookShelfIterator。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/1Iterator模式/2.jpg)

# 为什么要使用Iterator模式

在示例程序中，具体的集合类BookShelf底层是以数组存储元素的，那么为何不像开篇介绍的那样，直接使用for循环遍历呢？

一个重要的原因就是，引入了Iterator模式后，我们便可以将遍历与实现分离开来。

举个例子，假如我们将BookShelf中的books字段的权限放开，那么此时可以这样遍历：

```
for (int i = 0; i < books.length; i++)
    System.out.println(books[i].getName());
```

这样确实代码量更少，BookShelf也不用费心设计什么属于自己的迭代器了。但是这样做的问题在于，一旦我们改变了BookShelf底层的存储方式，比如我们改用链表作为数据的存储结构，那么调用者就必须要了解BookShelf内部的数据结构，然后做出相应的改动。

反观我们在示例程序中的迭代方式：

```
Iterator<Book> iterator = bookShelf.iterator();
while (iterator.hasNext())
    System.out.println(iterator.next().getName());
```

对于这种迭代方式而言，不论BookShelf底层的存储结构如何修改，只要它还实现Aggregate接口，即提供符合Iterator接口约束条件的迭代器，那么对于迭代操作而言，调用者的代码就无需做任何修改，也不需要关注BookShelf到底做了什么样的改动。

举一个实际的例子，books作为一个数组，一旦设定了maxsize就不可扩展了。假如我们现在需要书架是可扩展的了，那么我们就可以考虑使用ArrayList来作为底层存储结构。修改后的BookShelf类为：

```
package design1;

import java.util.ArrayList;
import java.util.List;
import java.util.NoSuchElementException;

public class BookShelf<E> implements Aggregate<E> {

    private List<Book> books;

    public BookShelf(int initsize) {
        if (initsize < 0)
            throw new IllegalArgumentException("bookShelf 's initsize < 0");
        this.books = new ArrayList<Book>(initsize);
    }

    public void appendBook(Book book) {
        if (null == book)
            throw new NullPointerException("book is null");
        this.books.add(book);
    }

    @Override
    public Iterator<E> iterator() {
        return new BookShelfIterator<E>(this);
    }

    int getLength() {
        return this.books.size();
    }

    Book getBookAt(int index) {
        if (index < 0 || index >= books.size())
            throw new IllegalArgumentException("index < 0 || index >= bookShelf 's size");
        return this.books.get(index);
    }
}
```

这样书架就变成可扩展的了。Main.java的输出保持不变，此时即便添加了超出初始容量的书籍，代码也不会报错了。值得注意的是，因为此前books就没有暴露给BookShelfIterator，因此BookShelfIterator虽然是BookShelf的附属迭代器，但是我们同样无需修改它，只需要修改BookShelf就可以啦。

更进一步的来讲，即便我们将BookShelf整体的换掉，例如，我们不使用书架来存放书了，而改用书包，即Schoolbag类来存放书本，那么对于这段迭代调用的代码而言，依然无需做任何改动，调用者也无需了解Schoolbag到底是个什么东西，只要Schoolbag依然还实现着Aggregate接口，即还是一个符合约束的集合就行。这也是为什么我们在迭代时使用Iterator作为引用，而非BookShelfIterator的原因。其实，并不仅仅是Iterator模式，对于所有设计模式而言，使用抽象的抽象类和接口，而不是具体的实现类作为组件间沟通的桥梁是至关重要的(六原则中的[依赖倒转原则]())。

这在设计模式领域有着非凡的意义。因为设计模式的一个非常重要的目标就是编写"可复用"的"高内聚低耦合"的组件。这样一旦某一个组件发生变化，那么被它影响的组件将尽可能少，且即便是被它影响的组件，需要改动的地方也会尽可能的少(六原则中的[单一职责原则]())。

# Java集合框架对Iterator模式的应用

Java的集合框架使用Iterator模式完成对集合中的元素的迭代。下面我们以具体的实现类ArrayList为例，来看看它到底是怎么做的。

下面给出类图。类图中只记述与迭代操作相关的字段及方法：

![3.jpg](/images/blog_pic/Java 设计模式/1Iterator模式/3.jpg)

# 相关设计模式

**[13.Visitor模式]()**

从广义的角度来讲，Iterator模式是Visitor模式的特例。

Visitor模式的目的是将处理自数据结构中分离出来，而Iterator模式的目的则是将迭代操作自容器结构中分离出来。