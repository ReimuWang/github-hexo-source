---
title: Java JDK7源码-java.util.RandomAccess
date: 2017-06-22 18:17:33
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public interface RandomAccess {
}
```

# 综述

本接口是Java集合框架中的一员，是被List所使用的标记接口，实现该接口表明List支持快速(通常是指常数时间)随机访问。

本接口的主要目的为在随机或顺序访问list时允许通用算法修改其行为以提供更好的性能。

当把基于随机访问list(例如ArrayList)的最佳算法用于顺序访问list(例如LinkedList)时，算法在用时上会呈平方阶增长。因此在设计list访问的通用算法时，最好先看一下传入的list是否实现了本接口，若未实现本接口，则说明传入list天然不支持随机访问，在设计算法时必须特殊考虑以保证访问的性能在可接受的范围内。

随机访问与顺序访问的界限通常是模糊的。例如，一些List在长度巨大时的访问时间复杂度接近于线性阶，但是在实际使用中，长度一般时的时间复杂度为常数阶。这样的List通常也会实现本接口。

一般来说，当某个list执行循环1的效率高于循环2时，即认为应该实现本接口。

循环1：

```
for (int i=0, n=list.size(); i < n; i++)
    list.get(i);
```

循环2：

```
for (Iterator i=list.iterator(); i.hasNext(); )
    i.next();
```

换句话说，在欲迭代一个list之前，最好先判断其是否已实现了本接口。然后采取不同的调用方法：

```
if (list instance of RandomAccess) {
    for (int i = 0; i < list.size(); i++){
        // TODO
    }
} else {
    Iterator iterator = list.iterator();
    while (iterator.hasNext()) {
        // TODO
    }
}
```

以下类是对性能的测试：

```
import java.util.ArrayList;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.List;

import org.junit.Test;

public class TestRandomAccess {

    private final static int LIST_SIZE = 1000;

    private final static int TEST_COUNT = 1000; 

    @Test
    public void testRandomAccess() {
      List<Integer> arraylist = new ArrayList<Integer>();
      for (int i = 0; i < TestRandomAccess.LIST_SIZE; i++)  arraylist.add(i);
      List<Integer> linkedList = new LinkedList<Integer>();
      for (int i = 0; i < TestRandomAccess.LIST_SIZE; i++)  linkedList.add(i);

      System.out.println("size=" + TestRandomAccess.LIST_SIZE + ",test count=" + TestRandomAccess.TEST_COUNT + "\n");
      System.out.println("ArrayList for time=" + this.forTime(arraylist));
      System.out.println("ArrayList iterator time=" + this.iteratorTime(arraylist));
      System.out.println("LinkedList for time=" + this.forTime(linkedList));
      System.out.println("LinkedList iterator time=" + this.iteratorTime(linkedList));
    }

    private long forTime(List<Integer> list) {
        long beginTime = System.currentTimeMillis();
        for (int count = 0; count < TestRandomAccess.TEST_COUNT; count++) {
            for (int i = 0; i < list.size(); i++) list.get(i);
        }
        return System.currentTimeMillis() - beginTime;
    }

    private long iteratorTime(List<Integer> list) {
        long beginTime = System.currentTimeMillis();
        for (int count = 0; count < TestRandomAccess.TEST_COUNT; count++) {
            Iterator<Integer> iterator = list.iterator();
            while (iterator.hasNext()) iterator.next();
        }
        return System.currentTimeMillis() - beginTime;
    }
}
```

运行结果为：

```
size=1000,test count=1000

ArrayList for time=7
ArrayList iterator time=12
LinkedList for time=466
LinkedList iterator time=15
```