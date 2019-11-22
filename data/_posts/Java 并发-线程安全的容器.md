---
title: Java 并发-线程安全的容器
date: 2018-01-24 11:19:36
tags: [Java,并发]
categories: Java 并发
---

为了降低开发人员进行并发编程的工作成本，Java API提供了常见的线程安全的容器(链表，Map，队列等)。这些容器大多位于java.util.concurrent包中(其他位置也零星的分布着线程安全的容器，例如java.util.Vector，这些容器的性能往往较差)，常用的列举如下：

- java.util.concurrent.ConcurrentHashMap: 可以看作线程安全的高效并发的java.util.HashMap。

- java.util.concurrent.CopyOnWriteArrayList: 顾名思义，该容器属于java.util.ArrayList一系。在读多写少的场合，它的性能远高于java.util.Vector。详见[Java 并发-CopyOnWriteArrayList](/2018/02/07/Java 并发-CopyOnWriteArrayList/)。

- java.util.concurrent.ConcurrentLinkedQueue: 使用链表实现，可以看作线程安全的高效并发的LinkedList。详见[Java 并发-ConcurrentLinkedQueue](/2018/02/05/Java 并发-ConcurrentLinkedQueue/)。

- java.util.concurrent.BlockingQueue: java.util.Queue接口的子接口，是线程安全的阻塞队列，常用作并发编程中的缓冲区。Java API提供了它的数组，链表等具体的实现。详见[Java 并发-BlockingQueue](/2018/02/07/Java 并发-BlockingQueue/)。

- java.util.concurrent.ConcurrentSkipListMap: 这是一个底层以Map实现的线程安全的跳表，可进行高效的查找。详见[Java 并发-ConcurrentSkipListMap](/2018/02/07/Java 并发-ConcurrentSkipListMap/)。

<!-- more -->

纵观上述例子，我们几乎都可以找到一个统一的关键词，"高效"。这份高效源于专业：上述容器是专门为并发编程设计的，自然更能适应并发环境。

在此，我们不妨顺着"专业"继续往下说：专业同样意味着不全面。举个例子，我们在串行环境中创建了一个HashMap的实例，如果我们想将它转换为ConcurrentHashMap是比较麻烦的：只能新建ConcurrentHashMap的实例，然后将已有的hashMap中的数据一点点的导入。对此，Java API中已提供了解决方法：如果我们对并发容器的性能要求不高，那么其实是可以通过一个方法简单的将它们线程安全化的，这些方法位于java.util.Collections工具类中，可参见[Java 并发-容器线程安全化方法](/2018/02/01/Java 并发-容器线程安全化方法/)。