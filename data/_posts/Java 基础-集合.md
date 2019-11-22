---
title: Java 基础-集合
date: 2017-09-29 16:25:49
tags: [Java,集合,Collection]
categories: Java 基础
---

# 阐述ArrayList、Vector、LinkedList的存储性能和特性。

<!-- more -->

ArrayList 和Vector都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector中的方法由于添加了synchronized修饰，因此Vector是线程安全的容器，但性能上较ArrayList差，因此已经是Java中的遗留容器。LinkedList使用双向链表实现存储（将内存中零散的内存单元通过附加的引用关联起来，形成一个可以按序号索引的线性结构，这种链式存储方式与数组的连续存储方式相比，内存的利用率更高），按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。Vector属于遗留容器(Java早期的版本中提供的容器，除此之外，Hashtable(可简单的看作是线程安全的HashMap)、Dictionary、BitSet、Stack、Properties都是遗留容器)，已经不推荐使用，但是由于ArrayList和LinkedListed都是非线程安全的，如果遇到多个线程操作同一个容器的场景，则可以通过工具类Collections中的synchronizedList方法将其转换成线程安全的容器后再使用(这是对装潢模式的应用，将已有对象传入另一个类的构造器中创建新的对象来增强实现)。

补充：遗留容器中的Properties类和Stack类在设计上有严重的问题，Properties是一个键和值都是字符串的特殊的键值对映射，在设计上应该是关联一个Hashtable并将其两个泛型参数设置为String类型，但是Java API中的Properties直接继承了Hashtable，这很明显是对继承的滥用。这里复用代码的方式应该是Has-A关系而不是Is-A关系，另一方面容器都属于工具类，继承工具类本身就是一个错误的做法，使用工具类最好的方式是Has-A关系(关联)或Use-A关系(依赖)。同理，Stack类继承Vector也是不正确的。

# Collection和Collections的区别？

Collection是一个接口，它是Set、List等容器的父接口；Collections是个一个工具类，提供了一系列的静态方法来辅助容器操作，这些方法包括对容器的搜索、排序、线程安全化等等。

# List、Map、Set三个接口存取元素时，各有什么特点？

List以特定索引来存取元素，可以有重复元素。Set不能存放重复元素(用对象的equals()方法来区分元素是否重复)。Map保存键值对(key-value pair)映射，映射关系可以是一对一或多对一。Set和Map容器都有基于哈希存储和排序树的两种实现版本，基于哈希存储的版本理论存取时间复杂度为O(1)，而基于排序树版本的实现在插入或删除元素时会按照元素或元素的键(key)构成排序树从而达到排序和去重的效果。

# TreeMap和TreeSet在排序时如何比较元素？Collections工具类中的sort()方法如何比较元素？

TreeSet(可以简单的看作有序的集合)要求存放的对象所属的类必须实现Comparable接口，该接口提供了比较元素的compareTo()方法，当插入元素时会回调该方法比较元素的大小。TreeMap(可以简单的看作有序的Map)要求存放的键值对映射的键必须实现Comparable接口从而根据键对元素进行排序。Collections工具类的sort方法有两种重载的形式，第一种要求传入的待排序容器中存放的对象比较实现Comparable接口以实现元素的比较；第二种不强制性的要求容器中的元素必须可比较，但是要求传入第二个参数，参数必须实现Comparator接口(需要重写compare方法实现元素的比较)，相当于一个临时定义的排序规则，其实就是通过接口注入比较元素大小的算法，也是对回调模式的应用(Java中对函数式编程的支持)。

```
import java.util.Set;
import java.util.TreeSet;

public class Test {
 
    public static void main(String[] args) throws Exception {
        Set<Person> set = new TreeSet<>();    // Java7的钻石语法(构造器后面的尖括号中不需要写类型)
        set.add(new Person("博丽灵梦", 16));
        set.add(new Person("八云紫", 999999999));
        set.add(new Person("雾雨魔理沙", 15));
        set.add(new Person("十六夜咲夜", 18));
        for(Person person : set) System.out.println(person);
    }
}

class Person implements Comparable<Person> {

    private String name;

    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student [name=" + name + ", age=" + age + "]";
    }

    @Override
    public int compareTo(Person o) {
        return this.age - o.age;    // 比较年龄(年龄的升序)
    }
}
```

输出：

```
Student [name=雾雨魔理沙, age=15]
Student [name=博丽灵梦, age=16]
Student [name=十六夜咲夜, age=18]
Student [name=八云紫, age=999999999]

```

```
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class Test {

    public static void main(String[] args) throws Exception {
        List<Person> list = new ArrayList<>();     // Java 7的钻石语法(构造器后面的尖括号中不需要写类型)
        list.add(new Person("博丽灵梦", 16));
        list.add(new Person("八云紫", 999999999));
        list.add(new Person("雾雨魔理沙", 15));
        list.add(new Person("十六夜咲夜", 18));

        // 通过sort方法的第二个参数传入一个实现Comparator接口的对象
        // 相当于是传入一个比较对象大小的算法到sort方法中
        // 由于Java中没有函数指针、仿函数、委托这样的概念
        // 因此要将一个算法传入一个方法中唯一的选择就是通过接口回调
        Collections.sort(list, new Comparator<Person> () {
 
            @Override
            public int compare(Person o1, Person o2) {
                return o1.age - o2.age;    // 比较年龄(年龄的升序)
            }
        });

        for(Person person : list) System.out.println(person);
    }
}

class Person {

    private String name;

    public int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student [name=" + name + ", age=" + age + "]";
    }
}
```

输出：

```
Student [name=雾雨魔理沙, age=15]
Student [name=博丽灵梦, age=16]
Student [name=十六夜咲夜, age=18]
Student [name=八云紫, age=999999999]
```