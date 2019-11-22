---
title: Java 并发-ConcurrentLinkedQueue
date: 2018-02-05 10:07:36
tags: [Java,并发]
categories: Java 并发
---

java.util.concurrent.ConcurrentLinkedQueue可以看作线程安全的高效并发的java.util.LinkedList，它应该是高并发环境下Java API提供的性能最高的队列了。它的类定义为：

```
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E> implements Queue<E>, java.io.Serializable
```

# 节点

顾名思义，ConcurrentLinkedQueue底层是以链表实现的。而作为一个链表，最核心的数据结构自然是构成链表的节点了。ConcurrentLinkedQueue的节点是其自身的静态成员内部类，该节点类的全部代码如下：

<!-- more -->

```
private static class Node<E> {
    // item,next支撑起了Node作为链表的节点的基础
    volatile E item;    // 节点中存储的数据值
    volatile Node<E> next;    // 下一个节点

    Node(E item) {
        NSAFE.putObject(this, itemOffset, item);
    }

    /**
     * 设置节点中存储的数据值
     * 利用CAS保证线程安全性：
     * 设置时当前值与期望值相同则设置为目标值
     * @param cmp E, 期望值
     * @param val E, 目标值
     * @return boolean, true--设置成功，false--设置失败
     */
    boolean casItem(E cmp, E val) {
        return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
    }

    void lazySetNext(Node<E> val) {
        UNSAFE.putOrderedObject(this, nextOffset, val);
    }

    /**
     * 设置本节点的下一个节点
     * 利用CAS保证线程安全性：
     * 设置时当前值与期望值相同则设置为目标值
     * @param cmp Node<E>, 期望值
     * @param val Node<E>, 目标值
     * @return boolean, true--设置成功,false--设置失败
     */
    boolean casNext(Node<E> cmp, Node<E> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }


    private static final sun.misc.Unsafe UNSAFE;
    private static final long itemOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class k = Node.class;
            itemOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("item"));
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

# head与tail

ConcurrentLinkedQueue类中记录了两个特殊的节点：

```
private transient volatile Node<E> head;

private transient volatile Node<E> tail;
```

head指向队首节点，tail指向队尾节点。它们都不为null，但其中的item可能为null。

在ConcurrentLinkedQueue内部，可通过ConcurrentLinkedQueue.succ()完成队列自特定节点起的遍历：

```
final Node<E> succ(Node<E> p) {
    Node<E> next = p.next;
    return (p == next) ? head : next;
}
```

关于这个方法，需要注意的有两点：

1. 该方法并没有任何的并发安全控制，但由于它的访问权限为默认，因此在调用它的地方，如果必要的话，应该都会加上相应的线程安全措施。

2. p == next代表next指向其本身的节点，它们被称为"哨兵节点"。这种节点是指那些原则上已不存在，却因为各种原因(例如并发安全问题)暂时还未来得及删掉的节点：例如要删除的节点，或者空节点。此时因已无法通过哨兵节点拿到next，故只能返回head从头再来。

按照定义，我们理所当然的会认为head指向队列的第一个节点，tail指向队列的最后的一个节点。但实际上，如果我们将链表中一次取next操作视为时间复杂度计算中的基本步骤的话，它们的准确定义为：

- head:以该节点为起点，遍历至实际有效的第一个节点的时间复杂度为O(1)。

- tail:以该节点为起点，遍历至实际有效的最后一个节点的时间复杂度为O(1)。

Oh,No!这是何等操蛋又让人迷茫的定义啊。我们不妨以从零开始逐个插入节点为例来加深一下蛋疼感：

当插入元素为列表的第奇数个节点时，tail不会实际移动，此时tail指向得是倒数第二个节点；当插入元素为列表的第偶数个节点时，tail会连续移动过两个节点，指向队列的最后一个节点。

![0.png](/images/blog_pic/Java 并发/ConcurrentLinkedQueue/0.png)

弹出节点的套路也是类似：若被弹出的节点是奇数个节点时，head不会实际移动，此时head指向得是第一个节点的前一个节点，也就是一个已实际上被移除的节点；若被弹出的节点是偶数个节点时，head会连续移动两次，指向队列的第一个节点。

![1.png](/images/blog_pic/Java 并发/ConcurrentLinkedQueue/1.png)

......额......

为什么这么搞暂时不知，姑且记下，有空详查。

# offer()方法

作为一个队列，最重要的自然就是offer()及poll()方法了。先来看offer的源码：

```
public boolean offer(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);

    // 该方法的核心目标就是找到真正的最后一个节点，而后将e插入到它的后面
    for (Node<E> t = tail, p = t;;) {
        // 由于tail未必是真正的最后一个节点，因此第一次循环开始前设置的p=tail也未必是最后一个节点
        // 然而tail必然可以在O(1)时间内找到真正的最后一个节点，因此for循环的定义体中并未设置出口：找到前一直循环下去，总会出去的
        Node<E> q = p.next;
        if (q == null) {
            // q==null说明p当真是最后一个节点了
            if (p.casNext(null, newNode)) {
                // 成功将p.next设置为e。此时插入动作已完成
                // 下面的这个if是专门用来处理tail奇数个节点时不动偶数个节点时连动两次的逻辑的
                if (p != t)
                    // p!=t说明上一个节点是奇数个节点，那么本节点就是偶数个节点了，要连动两次
                    // 并未针对casTail()的返回值做处理。也就是说不管tail更新成功与否都无妨
                    // 不得不说这里的设计思路屌爆了，给大神点赞：
                    // 设置成功：没什么说的，成了。
                    // 设置失败：失败说明在设置时又有新节点插入了，那么本节点自然也就不可能是tail了，自然失败就失败了
                    casTail(t, newNode);
                // 唯一出口，也就是说本方法是不可能返回false的=-=
                return true;
            }
            // p.next设置为e失败了，再重来一次。
            // 因该分支中并未修改p的值，因此再次重来p还是进入本if之前的那个p
        }
        else if (p == q)
            // p是哨兵节点
            // 首先不管怎么样，经过下面的语句后t都会被修正为t=tail
            // t是在for循环开始前设置等于tail的，串行环境下应始终有t=tail才是
            // 如果执行下面语句前t都不等于tail了，说明这个for循环的根基已被动摇，只能从head开始遍历
            // 如果将本方法的思路视为乐观锁，那么从head开始遍历实在是最糟糕的情况了
            // 如果执行下面语句前依然有t=tail，说明本for循环等根基还在，可以从t(也就是此时最新的tail)再次来过
            p = (t != (t = tail)) ? t : head;
        else
            // q并非最后一个节点，还要再向后遍历
            // 简单来看，直接设置p=q即可，这样便可实现向后遍历
            // 不过大神提到了一种特殊情况：p不是t了(也就是p已经移动过了)且t不是tail了(前面已经提到，串行环境下是不可能的。显然是出现了并发问题，动摇了本for循环的根基)
            // 此时要从t开始遍历，依然是乐观锁思想下的悲观情况
            p = (p != t && t != (t = tail)) ? t : q;
    }
}
```

代码其实并不长，但却不大好理解(我写的注释比代码还多)。不过简单看一下就可以发现：该方法并没有使用synchronized或是Lock，它只调用了两个方法casNext(),casTail()，而这两个方法也只用到了CAS。

唔，=-=。。。

这说明了什么！这说明了什么！！这说明了什么！！！

这说明ConcurrentLinkedQueue是一个无锁的队列，它只用底层的CAS就保证了线程安全性！

难怪doc都说ConcurrentLinkedQueue是高并发环境下Java API提供的性能最高的队列，给Doug Lea爸爸跪了。

需要强调一下的是：t!=t并不是原子操作，也就说是先取一次t存起来，然后再取第二次t，而后比较这两次取得的t的结果。因此在并发环境中，t!=t返回true是完全有可能。我们不妨以一个小例子来证明一下：

```
public class Test {

    public static void main(String[] args) {
        Object o1 = new Object();
        System.out.println(o1 != o1);
    }
} 
```

我们用javap反编译一下：

```
Classfile /D:/Test.class
  Last modified 2018-2-6; size 487 bytes
  MD5 checksum 76721705c54f8bd636869e0cfab7f65e
  Compiled from "Test.java"
public class Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #2.#18         //  java/lang/Object."<init>":()V
   #2 = Class              #19            //  java/lang/Object
   #3 = Fieldref           #20.#21        //  java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #22.#23        //  java/io/PrintStream.println:(Z)V
   #5 = Class              #24            //  Test
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               main
  #11 = Utf8               ([Ljava/lang/String;)V
  #12 = Utf8               StackMapTable
  #13 = Class              #25            //  "[Ljava/lang/String;"
  #14 = Class              #19            //  java/lang/Object
  #15 = Class              #26            //  java/io/PrintStream
  #16 = Utf8               SourceFile
  #17 = Utf8               Test.java
  #18 = NameAndType        #6:#7          //  "<init>":()V
  #19 = Utf8               java/lang/Object
  #20 = Class              #27            //  java/lang/System
  #21 = NameAndType        #28:#29        //  out:Ljava/io/PrintStream;
  #22 = Class              #26            //  java/io/PrintStream
  #23 = NameAndType        #30:#31        //  println:(Z)V
  #24 = Utf8               Test
  #25 = Utf8               [Ljava/lang/String;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               java/lang/System
  #28 = Utf8               out
  #29 = Utf8               Ljava/io/PrintStream;
  #30 = Utf8               println
  #31 = Utf8               (Z)V
{
  public Test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class java/lang/Object
         3: dup           
         4: invokespecial #1                  // Method java/lang/Object."<init>":()V
         7: astore_1      
         8: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: aload_1       
        12: aload_1       
        13: if_acmpeq     20
        16: iconst_1      
        17: goto          21
        20: iconst_0      
        21: invokevirtual #4                  // Method java/io/PrintStream.println:(Z)V
        24: return        
      LineNumberTable:
        line 4: 0
        line 5: 8
        line 6: 24
      StackMapTable: number_of_entries = 2
           frame_type = 255 /* full_frame */
          offset_delta = 20
          locals = [ class "[Ljava/lang/String;", class java/lang/Object ]
          stack = [ class java/io/PrintStream ]
           frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/Object ]
          stack = [ class java/io/PrintStream, int ]

}
```

我们来看main方法，o1于指令7被存入局部变量表索引为1的位置。然后11,12连取了两次，随后13行才开始计较。很显然，在并发环境中，11,12两行的取值操作并非线程安全的。

# poll()方法

代码如下：

```
public E poll() {
    // 这里用了因为不太好驾驭以及可能导致代码混乱而不被推荐的goto
    // ...
    // 大神这么屌当然是可以驾驭的，所以没毛病
    restartFromHead:
        for (;;) {
            for (Node<E> h = head, p = h, q;;) {
                // 由于head未必是真正的第一个节点，因此第一次循环开始前设置的p=head也未必是第一个节点
                // 然而head必然可以在O(1)时间内找到真正的第一个节点，因此for循环的定义体中并未设置出口：找到前一直循环下去，总会出去的
                E item = p.item;

                if (item != null && p.casItem(item, null)) {
                    // 进入此if有两个条件：
                    // 1.item!=null，说明找到了，p就是第一个节点
                    // 2.设置p.item为null完成。
                    // 此时第一个元素已从队列中移除。核心操作已完成
                   // 下面的这个if是专门用来处理head奇数个节点时不动偶数个节点时连动两次的逻辑的
                    if (p != h)
                        // 该方法返回值为空，内部稍显复杂，并不像offer()方法时的casTail()那么简单，就不深入分析了
                        updateHead(h, ((q = p.next) != null) ? q : p);
                    // 返回被移除元素的值
                    return item;
                }
                else if ((q = p.next) == null) {
                    // 准备进入本if的判定说明上一个if没过，也就是说p此时是指在第一个节点前面的某个废弃节点上的
                    // 再说本if的判定内容，p的下一个节点为null说明就没第一个节点了(因为p此时必是指在废弃节点上的)
                    updateHead(h, p);
                    return null;
                }
                else if (p == q)
                    // 进入本if的判定说明上一个if没过，此时q=p.next
                    // 若有p=q则说明p是哨兵，触发了乐观锁思路下的悲观情况，goto到最重头再来
                    continue restartFromHead;
                else
                    // 前面的if都过了，说明
                    // 1.p是指在第一个节点前的某个废弃节点上的
                    // 2.p的下一个节点不为空(也就是说起码当前是看不出来队列是否为空的)
                    // 3.p不是哨兵
                    // 那么很自然的，就继续向后遍历了
                    p = q;
            }
        }
    }
```

# 总结

虽然本文只是简单的介绍了ConcurrentLinkedQueue的offer()及poll()方法，但我们已经能体会到只用CAS实现无锁的线程安全容器的困难程度了：大神写好的代码读着都费劲，更别说自己去设计了。因此虽然无锁的性能确实很高，但在一般的程序开发中的应用却极少。