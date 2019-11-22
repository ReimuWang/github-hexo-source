---
title: Java 并发-SynchronousQueue
date: 2018-05-22 19:03:36
tags: [Java,并发,SynchronousQueue,无锁]
categories: Java 并发
---

在[Java 并发-线程池](/2017/10/03/Java 并发-线程池/)中，我们提到了一个线程池常用的阻塞队列：SynchronousQueue。当时对它的描述很简略，在这里再次摘录如下：

**SynchronousQueue**

即直接提交队列。其实这已经不能算作一种队列了：它并没有容量，即容量为0。换句话说，任何一个对SynchronousQueue的写需要等待一个对SynchronousQueue的读，反之亦然。因此，SynchronousQueue与其说是一个队列，更像是一个数据的中转站。

<!-- more -->

虽然当时没有细说，不过这实在是一个很奇特的队列了，那么它是如何实现的呢？

SynchronousQueue内部使用了大量的无锁(CAS)操作来进行并发控制，作为一个队列，外部最关心也最常用的自然就是它的put()和take()方法了：

```
public void put(E o) throws InterruptedException {
    if (o == null) throw new NullPointerException();
    if (transferer.transfer(o, false, 0) == null) {
        Thread.interrupted();
        throw new InterruptedException();
    }
}

public E take() throws InterruptedException {
    Object e = transferer.transfer(null, false, 0);
    if (e != null)
        return (E)e;
    Thread.interrupted();
    throw new InterruptedException();
}
```


实在是有趣！两个功能完全相反的方法内部调用的核心方法居然是同一个(正所谓殊途同归)，即transferer.transfer()方法。而transferer是SynchronousQueue的成员变量：

```
private transient volatile Transferer transferer;
```

而Transferer是SynchronousQueue的静态内部类，它的代码很短，全部代码如下：

```
abstract static class Transferer {
    /**
     * @param e, 若e==null，说明这是一次take()操作
     *           若e!=null，说明这是一次put()操作
     * @param timed, 是否存在timeout时间
     * @param nanos, 在timed==true的前提下，表示timeout的最大容忍时长
     * @return 为null表示本次操作失败(超时或者中断)
     *         反之操作成功：take()时为取到的元素；put()时为新放入的元素
     */
    abstract Object transfer(Object e, boolean timed, long nanos);
}
```

这是一个抽象类，要想真的能new出实例，自然还需要非抽象的子类，我们来看一下SynchronousQueue的transferer是怎么new的，按照我们已有的经验，这种重要的成员变量的初始化一般都会在构造函数中，因此我们不妨看一下SynchronousQueue的构造函数：

```
public SynchronousQueue() {
    this(false);
}

/**
 * fair, true - 线程等待队列为公平队列，即遵循FIFO
 *       false - 线程排序不定
 */
public SynchronousQueue(boolean fair) {
    transferer = fair ? new TransferQueue() : new TransferStack();
}
```

这里提到了一个线程等待队列，后文将详述。很显然，SynchronousQueue针对这个线程等待队列是否公平设计了两种实现，我想这就是为什么Transferer被设计为抽象类的原因吧，显然，这两个类都应是Transferer的子类。这两个类也均为SynchronousQueue的静态内部类，这里只贴出它们的类定义：

```
static final class TransferQueue extends Transferer

static final class TransferStack extends Transferer
```

接下来，就让我们详细说说这个线程等待队列吧！

简单来说，我们可以认为SynchronousQueue实际上维护的就是这个线程等待队列，只不过它因为一些限制，使得外部观测者认为这个队列的容量为零。

我们可以这样认为，线程等待队列中的元素是线程的请求。我们不妨举一个小例子，假设有如下线程等待队列：

- **元素1：**线程1请求写入数据1
- **元素2：**线程2请求写入数据2
- **元素3：**线程3请求写入数据3
- **元素4：**线程4请求写入数据4
- **元素5：**线程4请求写入数据5

此时来了一个新的请求，例如线程5请求写入数据5。它与队首元素1的请求类型相同，不符合SynchronousQueue的基本规定(任何一个对SynchronousQueue的写需要等待一个对SynchronousQueue的读，反之亦然)，线程5就会被存入线程等待队列的队首，

- **元素1：**线程5请求写入数据5
- **元素2：**线程1请求写入数据1
- **元素3：**线程2请求写入数据2
- **元素4：**线程3请求写入数据3
- **元素5：**线程4请求写入数据4

然后又来了一个新的请求：线程6请求读取数据。这与队首的元素1就不冲突了。此时元素1会被弹出，构成元素1-新请求的请求对，然后处理这个请求对。经过该步，线程等待队列变为:

- **元素1：**完成状态请求对：线程5请求写入数据5-线程6请求读取数据
- **元素2：**线程1请求写入数据1
- **元素3：**线程2请求写入数据2
- **元素4：**线程3请求写入数据3
- **元素5：**线程4请求写入数据4

请求对处理完成后，线程等待队列变为：

- **元素1：**线程1请求写入数据1
- **元素2：**线程2请求写入数据2
- **元素3：**线程3请求写入数据3
- **元素4：**线程4请求写入数据4

介绍完核心思想，我们再以具体的实现TransferStack.transfer()为例，看看具体的源码是如何实现的：

```
Object transfer(Object e, boolean timed, long nanos) {

    SNode s = null;    // 封装当前请求的元素
    int mode = (e == null) ? REQUEST : DATA;

    for (;;) {
        SNode h = head;    // 线程等待队列中的队首元素
        if (h == null || h.mode == mode) {    // 队列为空，或队首元素与当前请求的类型相同(均为读，或均为写)
            if (timed && nanos <= 0) {    // 不进行等待
                if (h != null && h.isCancelled())    // 当前队首元素处于取消状态的话，帮助队首元素处理它的取消行为
                    casHead(h, h.next);
                else
                    return null;    // 直接判定为失败，新请求无法进入队列
            } else if (casHead(h, s = snode(s, e, h, mode))) {    // 将新请求生成为元素，并插入到队首且插入成功
                // awaitFulfill()方法会进行自旋，若自旋结束后也未出现新的匹配请求(即当前元素为读的话，出现了为写的请求；或当前元素为写的话，出现了读的请求)，则会挂起线程
                // 当匹配请求出现后，线程被唤醒
                SNode m = awaitFulfill(s, timed, nanos);
                if (m == s) {    // 其他线程已帮助该元素完成了它的工作，后续流程取消
                    clean(s);
                    return null;
                }
                // 继续向下走了，说明没有接到别人的帮助
                if ((h = head) != null && h.next == s)    // 帮助临近的后续元素
                    casHead(h, s.next);
                return (mode == REQUEST) ? m.item : s.item;
            }
        } else if (!isFulfilling(h.mode)) {    // 队首元素与当前请求的类型不同
            if (h.isCancelled())    // 若该队首元素已在其他线程的帮助下被取消了
                casHead(h, h.next);    // 弹出当前队首，并重试
            // 能进入下面的else if说明当前已能构成请求对
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {    // 以当前请求对生成一个新的元素
                                                                         // 将它置为完成态
                                                                         // 并置于队首
                for (;;) {    // 除非队列为空，否则一直检测匹配者
                    SNode m = s.next;    // 设置m(原队列队首)为s(当前请求元素)的匹配者
                    if (m == null) {    // 循环到最后也没找到匹配者
                        casHead(s, null);
                        s = null;
                        break;
                    }
                    SNode mn = m.next;
                    // tryMatch()会激活一个等待线程，并将m传递给那个线程
                    if (m.tryMatch(s)) {    // 设置成功，数据投递完成
                                            // s,m均可弹出了
                        casHead(s, mn);
                        return (mode == REQUEST) ? m.item : s.item;
                    } else    // 设置失败
                              // 说明已有其他线程帮助完成了该工作
                              // 弹出m即可
                        s.casNext(m, mn);
                }
            }
        } else {    // 头部元素恰好是完成态
                    // 说明此时恰有一个线程处于上一个else if中的匹配过程中
                    // 那么对于当前线程而言，比起处理目前手中已有的新进请求
                    // 更优先的事项是帮助其他线程完成这个完成态的元素
            SNode m = h.next;
            if (m == null)
                casHead(h, null);
            else {
                SNode mn = m.next;
                if (m.tryMatch(h))
                    casHead(h, mn);
                else
                    h.casNext(m, mn);
            }
        }
    }
}
```

值得关注的是，上文中有很多线程间互助的操作，例如：

- **第10行：**当前队首元素处于取消状态的话，帮助队首元素处理它的取消行为
- **第18行：**其他线程已帮助该元素完成了它的工作，后续流程取消
- **第23行：**帮助临近的后续元素
- **第56行：**更优先的事项是帮助其他线程完成这个完成态的元素

由此可知，在SynchronousQueue中，线程间不再仅仅只是竞争关系了，还体现着一种协作：线程进入线程等待队列意味着被挂起，那么那些没挂起的，还尚有余力的线程就可以帮助其他线程完成任务，这种模式可以更大程度上减少饥饿的可能，提高系统整体的并行性。