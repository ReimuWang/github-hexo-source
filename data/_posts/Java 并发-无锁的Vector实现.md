---
title: Java 并发-无锁的Vector实现
date: 2018-05-17 11:55:36
tags: [Java,并发,无锁,Vector]
categories: Java 并发
---

java.util.Vector是Java API提供的容器，它可以被看作是线程安全的java.util.ArrayList。该Vector是以synchronized来保证容器的线程安全性的。在[Java 并发-无锁](/2018/05/11/Java 并发-无锁/)中，我们介绍一些Java API中基础的，利用CAS做并发控制的小例子。下面，让我们跳出Java API，来分析一种第三方类库提供的Vector的无锁(即使用CAS做并发控制)实现。

<!-- more -->

该第三方jar包被称为Amino框架，它是一个采用无锁方式实现并行计算的框架。遗憾的是，我并未在Maven仓库或网上其他地方找到这个jar包，也无法找到它的源码。因此本文中的源码和分析完全来自于[实战Java高并发程序设计]一书。

在Amino框架中，该Vector被称作LockFreeVector。它的特点是可以根据需求动态扩展其内部的空间。其实际用于存储的数据结构为：

```
private final AtomicReferenceArray<AtomicReferenceArray<E>> buckets;
```

值得注意的是，该变量是final的，而Vector内存储的数据显然又需要是可变的，这就说明不变的仅仅是引用指向buckets这一点，buckets内部还是需要变化的。

另外，较之普通的Java API中的集合类的容器，buckets的数据结构是比较复杂的：它是一个数组，其元素又是一个数组(也就是我们常说的二维数组)。而且这些数组全部都是AtomicReferenceArray，这样从最底层的存储数据的容器开始，LockFreeVector就获得了一定的线程安全能力。

那么问题来了，我们都知道Vector实际存储的数据是一维的。那么为什么要用一个二维的数据结构呢？简单来说，这是为了便于容器的动态扩展(下文将进行详述，算法很巧妙，也很有趣)。我们可以稍稍回顾下AtomicReferenceArray内部用于存储数据的数据结构：

```
private final Object[] array;
```

它只是一个不可变的普通数组，如果只使用一维的AtomicReferenceArray的话，扩展将会非常麻烦。

此外，为了能对数组进行更有序的读写，还定义了一个名为Descriptor的类。它的作用是使用CAS操作写入新数据。

```
static class Descriptor<E> {

    /**
     * 整个Vector的长度
     */
    public int size;

    /**
     * 负责实际的写入
     */
    volatile WriteDescriptor<E> writeop;

    public Descriptor(int size, WriteDescriptor<E> writeop) {
        this.size = size;
        this.writeop = writeop;
    }

    /**
     * 写入
     */
    public void completeWrite() {
        WriteDescriptor<E> tmpOp = writeop;
        if (tmpOp != null) {
            tmpOp.doIt();
            writeop = null;
        }
    }
}

static class WriteDescriptor<E> {

    /**
     * 期望值
     */
    public E oldV;

    /**
     * 待写入的新值
     */
    public E newV;

    /**
     * 需修改的原子数组
     */
    public AtomicReferenceArray<E> addr;

    /**
     * 数组需修改的索引位置
     */
    public int addr_ind;

    public WriteDescriptor(AtomicReferenceArray<E> addr, int addr_ind, E oldV, E newV) {
        this.addr = addr;
        this.addr_ind = addr_ind;
        this.oldV = oldV;
        this.newV = newV;
    }

    public void doIt() {
        addr.compareAndSet(addr_ind, oldV, newV);
    }
}
```

在构造LockFreeVector的实例时，显然需要初始化它的核心组件buckets及descriptor：

```
public LockFreeVector() {
    buckets = new AtomicReferenceArray<AtomicReferenceArray<E>>(N_BUCKET);
    buckets.set(0, new AtomicReferenceArray<E>(FIRST_BUCKET_SIZE));
    // 在Descriptor外又包装了一层AtomicReference
    // 初始时Vector长度为0。未指定WriteDescriptor
    descriptor = new AtomicReference<Descriptor<E>>(new Descriptor<E>(0, null));
}
```

其中N_BUCKET及FIRST_BUCKET_SIZE均为常量值：

```
N_BUCKET = 30;
FIRST_BUCKET_SIZE = 8;
```

这里需要注意的是，buckets是final的，这意味着对于通过该构造函数生成的LockFreeVector实例而言，终其一生buckets的长度只能是30了。

同时，该构造函数还初始化了buckets的第一个元素，并将其的长度设为了8。而AtomicReferenceArray内部用于存储数据的数据结构为：

```
private final Object[] array;
```

它也是final的。这就说明第一个元素，也就是第一个数组的长度固定为8。

如果我们以此为依据推断的话，假设buckets的后续元素的长度也都是8，那么buckets岂不是只能存储240个数据吗？不管怎么说，这都实在是太少了点。

我们先来回顾下Vector的扩展逻辑，在不显式指定扩展规则的情况下，默认每次容量翻倍(多说一句，ArrayList无法显式指定，只能按默认方式扩展，默认每次容量变为原来的1.5倍)。其实LockFreeVector的思路也类似：buckets的第二维数组存储数据本身，第一维数组以冗余的方式存储扩展。后一个元素就是前一个元素下一次的扩展，扩展默认每次翻倍，也就是说，buckets中元素的长度其实是这样的：

```
元素0：长度2^3
元素1：长度2^4
元素2：长度2^5
...
元素29：长度2^32
```

因此，buckets能存储的元素总数其实是：

```
2^3 + 2^4 + 2^5 + ... + 2^32
```

这已经是一个非常巨大的数字了，通常的需求都可以满足。

这里还有一个需要注意的点，无论是Vector还是ArrayList，同一个时间点维护的容器只有1份，也就是说进行扩展后，上一个容器就丢掉了。之前已有的数据还要复制到新容器中，可用容量的上限也是新容器的长度。但是，对于LockFreeVector而言，每次扩展后，老容器不会被抛弃，仍可继续，此时就没有将老容器中的元素复制到新容器中的必要了。buckets容积的上限为所有已有容器的和。

当有一个新元素需加入LockFreeVector时，会调用它的push_back()方法。该方法会将待添加元素放到LockFreeVector的末尾。对于Vector或是ArrayList，这是一个很简单的操作，直接将新元素丢到最后一个元素之后即可，如果容量不够就扩展。但是对于LockFreeVector而言，问题就要稍微复杂一些了。在丢元素之前我们需要先确定新元素的位置：即它位于第几个数组的第几号索引。

前文我们介绍过负责写入的类Descriptor，它有一个名为size的字段。其含义是当前LockFreeVector已有元素的个数。我们会用它来计算新元素的位置。

我们先来确定新元素位于第几个数组。

我们不妨模拟一个情景，当前已使用了4个数组，且4个数组都恰好装满(也就是说，我们要找的LockFreeVector的最后一个元素就是第4个数组的最后一个元素)，我们不妨用二进制的int型来表示这4个数组的长度：

```
数组1：00000000 00000000 00000000 00001000 = 2 ^ 3， 28个前导零
数组2：00000000 00000000 00000000 00010000 = 2 ^ 4， 27个前导零
数组3：00000000 00000000 00000000 00100000 = 2 ^ 5， 26个前导零
数组4：00000000 00000000 00000000 01000000 = 2 ^ 6， 25个前导零
```

此时的size自然是这4个数字的和了，也就是

```
size = 00000000 00000000 00000000 01111000
```

如果我们想要引起1次进位，那么最小需要加的数字就是1000，也就是8了。加后值为：

```
size = 00000000 00000000 00000000 10000000
```

这刚好进入了第5个数组的领域，即新元素会位于第5个数组。

如果添加前元素没有装满呢？

因为此时已使用了4个数组，这就说明前3个数组均是装满了的，只有第4个数组还有空缺。在此前提下数组4已有元素肯定是小于它的最大容量的，即小于：

```
00000000 00000000 00000000 01000000
```

此时如果我们再求size，它的值肯定也是小于上文的最大值的，即小于：

```
00000000 00000000 00000000 01111000
```

如果我们依然按照上文的方式加二进制的1000，那么肯定是不会引起进位的。进而可以确认新元素依然还在第四个数组的范围内。

上文我们都是以最高位1为判断依据的，事实上，为了更为方便，我们还可以使用最高位1前有几个零来判断(本质上二者是一回事)。例如落于第4个数组中时前导零个数为25。落入第5个数组中时前导零个数为24。

确定了位于哪个数组后，我们再来确定新元素在该数组中的位置。

很显然，将最高位1置为0后，剩余数字就是新元素在数组中的位置。我们依然以第一个恰好填满4个数组的临界状态来分析。此时，新元素的加入导致了一次进位，即创建了一个新的数组，该新元素称为了新数组的第一个元素，即索引为0。我们还以第5个数组为例：

```
00000000 00000000 00000000 10000000
```

它共可容纳2^7个元素，此时如果我们不看代表第5个数组的最高位的这个1，它后面的这7位就是它内部元素的索引，且该规律不仅仅是针对初始为0000000时有效，只要没引起进位，直到1111111其实都是有效的。

推广下去，不管是第几个数组，其实都是这个道理。而如果数组没有装满，这不过是代表它在前面介绍的0000000-1111111增长变化的路上，依然符合规律。

说了这么多，我们总结以下使用size(现有元素数)求新元素位置的方法：

位于哪个数组：加二进制的1000后，根据前导零个数推断。
在数组中的索引：将最高位1置为0后的值。

分析结束，下面我们来看一下push_back()方法的源码：

```
public void push_back(E e) {
    Descriptor<E> desc;
    Descriptor<E> newd;
    do {
        desc = descriptor.get();
        desc.completeWrite();
        
        int pos = desc.size + FIRST_BUCKET_SIZE;
        int zeroNumPos = Integer.numberOfLeadingZeros(pos);
        int bucketInd = zeroNumFirst - zeroNumPos;
        if (buckets.get(bucketInd) == null) {
            int newLen = 2 * buckets.get(bucketInd - 1).length();
            if (debug)
                System.out.println("New Length is:" + newLen);
            buckets.compareAndSet(bucketInd, null, new AtomicReferenceArray<E>(newLen));
        }
        int idx = (0x80000000>>>zeroNumPos) ^ pos;
        newd = new Descriptor<E>(desc.size() + 1, new WriteDescriptor<E>(buckets.get(bucketInd), idx, null, e));
    } while (!descriptor.compareAndSet(desc, newd));
    descriptor.get().completeWrite();
}
```

最后，同理，让我们再来看下LockFreeVector的get()方法的源码。push_back()需要确定的是新元素需插入的位置，而get()需要确定的是待查询元素的位置：

```
@Override
public E get(int index) {
    int pos = index + FIRST_BUCKET_SIZE;
    int zeroNumPos = Integer.numberOfLeadingZeros(pos);
    int bucketInd = zeroNumFirst - zeroNumPos;
    int idx = (0x80000000>>>zeroNumPos) ^ pos;
    return buckets.get(bucketInd).get(idx);
}
```