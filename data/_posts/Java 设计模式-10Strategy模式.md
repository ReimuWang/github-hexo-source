---
title: Java 设计模式-10.Strategy模式
date: 2018-07-27 16:46:49
tags: [Java,设计模式]
categories: Java 设计模式
---

在《图解设计模式》一书中，Strategy模式被归入了第4部分[分开考虑]()。在GoF原书中，Strategy模式则被归入了[行为型设计模式]()。简单来说，Strategy模式可以被描述为：整体的，便捷的替换解决同一个问题的不同算法。

<!-- more -->

# 综述

strategy的含义是"策略，战略"。在编程领域，我们可以将其理解为算法。Strategy模式可以整体的替换算法的实现部分，能让我们依场景灵活的使用不同算法解决同一个问题。

# 示例程序

下面我们给出一个应用Strategy模式的小例子。它可以使用不同的算法进行排序。

首先给出类图：

![0.jpg](/images/blog_pic/Java 设计模式/10Strategy模式/0.jpg)

本程序中的所有代码将被统一置于design10包下，结构如下：

![1.jpg](/images/blog_pic/Java 设计模式/10Strategy模式/1.jpg)

其中Main.java是测试代码，Utils.java是工具类，并没有出现在类图中。

下面将逐个贴出每个类的源码。

**Utils类**

```
package design10;

public class Utils {

    private Utils() {}

    public static void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }

    public static void check(int[] a) {
        if (null == a)
            throw new NullPointerException("a is null");
    }
}
```

**Sort接口**

```
package design10;

public interface Sort {

    void sort(int[] a);
}
```

**BubbleSort类**

```
package design10;

public class BubbleSort implements Sort {

    @Override
    public void sort(int[] a) {
        Utils.check(a);
        for (int i = 0; i < a.length - 1; i++) {
            for (int j = 0; j < a.length - 1 - i; j++) {
                if (a[j] > a[j + 1]) Utils.swap(a, j, j + 1);
            }
        }
    }
}
```

**QuickSort类**

```
package design10;

public class QuickSort implements Sort {

    @Override
    public void sort(int[] a) {
        Utils.check(a);
        QuickSort.quickSort(a, 0, a.length - 1);
    }

    private static void quickSort(int[] a, int low, int high) {
        if (low >= high) return;
        int keyIndex = partition(a, low, high);
        quickSort(a, low, keyIndex - 1);
        quickSort(a, keyIndex + 1, high);
    }

    private static int partition(int[] a, int low, int high) {
        int left = low, right = high;
        int keyValue = a[low];
        while (left < right) {
            while (right > left && a[right] >= keyValue) right--;
            while (left < right && a[left] <= keyValue) left++;
            if (left < right) Utils.swap(a, left, right);
        }
        a[low] = a[left];
        a[left] = keyValue;
        return left;
    }
}
```

**Main类**

首先测试一下BubbleSort：

```
package design10;

import java.util.Arrays;


public class Main {

    public static void main(String[] args) {
        int[] a = {5, 1, 4, 3, 2, 1};
        Sort sort = new BubbleSort();
        sort.sort(a);
        System.out.println(Arrays.toString(a));
    }
}
```

输出：

```
[1, 1, 2, 3, 4, 5]
```

然后将算法整体替换为QuickSort：

```
package design10;

import java.util.Arrays;


public class Main {

    public static void main(String[] args) {
        int[] a = {5, 1, 4, 3, 2, 1};
        Sort sort = new QuickSort();
        sort.sort(a);
        System.out.println(Arrays.toString(a));
    }
}
```

输出：

```
[1, 1, 2, 3, 4, 5]
```

# 登场角色

上面的示例程序介绍了Strategy模式的Java实现，下面咱们试着跳出语言层面，抽象出Strategy模式中登场的角色。

**Strategy(抽象的策略)**

Strategy角色负责制定具体的策略必须实现的规范。在示例程序中，由Sort类扮演这个角色。

**ConcreteStrategy(具体的策略)**

ConcreteStrategy是实现了Strategy角色的约束的具体的策略。在示例程序中，由BubbleSort类及QuickSort类联袂扮演这个角色。

下面是抽象后，无关语言的类图：

![2.jpg](/images/blog_pic/Java 设计模式/10Strategy模式/2.jpg)

# 一些说明

很显然，Strategy模式与Java中的多态思想一脉相承。所不同的是，多态的重点在于"同一种行为的不同表现"。例如同样是"吃"，狗的吃与羊的吃所表现出的现象就完全不同。而Strategy模式则是"以不同的手段达成同一种表现"。

# 相关设计模式

**[3.Template Method模式]()**

Template Method模式与Strategy模式均与Java中多态的思想一脉相承。