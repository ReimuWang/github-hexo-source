---
title: Java 并发-并发级别
date: 2017-10-04 13:07:36
tags: [Java,并发]
categories: Java 并发
---

# 临界区

被多线程并发请求时可能导致糟糕结果的资源被称为临界区。最稳妥的做法是，同一时间点仅有一个线程能请求到临界区。

从狭义的角度上讲，临界区并不等同于被多线程并发请求的资源。临界区是其的子集。若被多线程并发请求的资源不可能因为并发访问产生不可预期的结果，那么程序员完全可以不将其设置为临界区。从这个角度而言，一个资源是否是被多线程并发请求的资源是客观事实，不因人的意志而转移；但是其是否是临界区则主要依托于人的判断：对于我而言，这个共享资源是否会因并发访问而产生不可预期的结果？

<!-- more -->

# 多线程的活跃性问题

下面我们就重点说下这个"糟糕"，从糟糕程度由重到轻，依次为：死锁(Deadlock) --> 饥饿(Starvation) --> 活锁(Livelock)。

**死锁**

简单的小例子：线程A持有资源x而欲请求资源y。线程B持有资源y而欲请求资源x。不巧的是x，y均为临界区。如果现状得不到改善(AB均不愿意放弃已到手的资源)，那么死锁将一直进行下去，永远看不到解决的希望。因此死锁是活跃性问题中最糟糕的情况：它会直接导致被锁住的线程卡死。

**饥饿**

分为两种情况：

- 线程优先级不够高，导致其始终无法获得临界区。在自然界中，母鸟给小鸟喂食时，强壮的小鸟总会更容易的抢到食物，而瘦弱的小鸟则不得不忍受饥饿。这也是并发中"饥饿"这个概念的出处。

- 已占有临界区的线程迟迟不肯让出临界区，导致其他线程无法获得该临界区。

与死锁不同，饥饿总还是有希望解除的，起码不能像断言死锁那样断定饥饿的话这个程序就没救了。但是饥饿仍然是有着残酷的竞争的：列举的两种情况说明该资源确实足够抢手，也就是所谓的"狼多肉少"。往乐观了想，强壮的小鸟吃饱了，不乐意抢了，瘦弱的小鸟就有机会吃上饭了；但是往悲观了想，若母鸟带来的食物并没有那么充足，或者那些强壮的小鸟是吃撑了也要继续下去的吃货，此时瘦弱的小鸟就有被饿死的可能。

**活锁**

举一个生活中的小例子：一个狭窄的过道，仅容两人通过。此时有两人相对而行又恰好走在同一边。于是二人均产生了给对方让路的行为。于是两人同时的让向另一边。于是二人又把刚才的剧情重演了一遍：又撞上了。通常情况下，如果这两个人情商正常的话，往往会进行简单的眼神或言语沟通，然后两人就能很好的通行了。

然而计算机并不具备这种在人类看来最简单的"情商"。如果程序设计不得当，两个线程会无休止的这样谦让下去。

之所以说"活锁"没有"饥饿"那么糟，是因为本质上来说，"活锁"中并不存在资源竞争：资源是足够的，仅仅只是程序的设计不当而已。

# 并发级别

并发级别描述的是面对上文中并发的缺陷，我们"该怎么办"，或者说"有何对策"。因此并发级别表达的是"请求方请求资源"时"请求方"的意愿。换成具体的情境，表达的就是"线程请求临界区"时线程的意愿及策略，属于主观愿望，也就是想不想的问题；而临界区中定义的"同一时间点仅有一个线程能请求到临界区"是临界区的客观情况，是能不能做得到的问题。线程在设定并发级别时如果足够"乐观"，那么便可能会任性的无视临界区的客观定义。

在应对并发问题的态度上，我们可以画一条数轴。负无穷大代表绝对的悲观：并发程序又复杂问题又多，咱们别并发了吧，如果实在要搞咱们就必须事无巨细把所有(注意是所有)可能的情况都catch住；正无穷大代表绝对的乐观：不要怂，就是干！问题，不存在的！

这种绝对的二元论的说法显然很极端，但是它们却为我们提供了标杆，我们在设计并发程序时总会偏向其中一方一点。简单来说，越是悲观对并发的检查就越严格；越是乐观对并发的检查就越宽松。从悲观到乐观排序的话并发级别大致有如下几类：阻塞(Blocking) --> 无饥饿(Starvation-Free) --> 无障碍(Obstruction-Free) --> 无锁(Lock-Free) --> 无等待(Wait-Free)。

**阻塞(Blocking)**

临界区同一时刻仅有一个线程能进入。其他请求该临界区的线程全部等着。简单粗暴，使得对临界区的访问又变回了串行的状态。

**无饥饿(Starvation-Free)**

在简单粗暴的阻塞的基础上，增加了对多线程的活跃性问题中的饥饿情况的处理(换句话说，无饥饿依然是阻塞的，其仅仅只是对饥饿的情况做出了改善)。假设当前正在使用Java的锁机制实现并发(synchronized是监视机制，并非锁。其也无法做到后文提到的公平锁)，如果说普通的阻塞级别的锁是非公平锁(判断线程执行与否仅看其优先级)的话，那么无饥饿实现的就是所谓的公平锁。最简单的公平锁是完全无视线程的优先级，而完全只按时间上的先后顺序执行，此时相当于用时间上的先后顺序作为新的优先级替换了线程原有的优先级。较为复杂的实现会把时间作为优先级的一个分量和线程原有的优先级进行融合：也就是综合考虑线程优先级和其已等待的时间。

我们常说的公平锁是由重入锁(ReentrantLock)所实现的，该锁属于最简单的那类公平锁。

**无障碍(Obstruction-Free)**

无障碍是检查最为严格的非阻塞调度。

先来说阻塞与非阻塞的区别。模拟一个场景，两个线程并发请求临界区。阻塞级别的并发(阻塞及无饥饿)会切实遵守临界区的客观规定，老老实实的等着。而非阻塞则是两个线程都进入临界区，没出问题最好，出了问题再想办法解决。而"解决策略"的不同就在非阻塞这个大的概念下又分化出了不同的并发级别。

如果真实情况是并发冲突的频率并不高，也即临界区中所描述的"被多线程并发请求，且有可能因此导致糟糕结果的资源"中的"可能"并不常出现，那么非阻塞是优于阻塞的；但是如果这个"可能"的出现概率很高，那么非阻塞时的"解决策略"相应的登场就更为频繁。无论设计得多为巧妙。这个"解决策略"也是决计比不上一开始便谨慎的进行悲观处理的阻塞调度的。此时非阻塞调度便劣于阻塞调度了。

相对于阻塞而言，非阻塞已经足够乐观了。但是在非阻塞内部却仍可细分。无障碍便是非阻塞中最为悲观的那一个。其"解决策略"多种多样，一种可行的方案是依赖于"一致性标记"。假如当前临界区的一致性标记值为0。当线程1进入临界区后，会先更新这个值，例如改为1，代表线程1已进入，而后线程在临界区中操作(无论是读操作还是写操作)，当其完事决定退出时会再看一眼这个标记，若仍为1，说明在此期间没有别的线程进入临界区，也就是满足"乐观"的预期，可以自然的退出；但是如果在此期间线程2也进入了临界区，那么它会在进入后将标记值更新为2。此时线程1在完事后回看时就会发现值对不上了，此时它便会采用fail-fast原则(不管新来的那个线程做了什么，也许人家只是读，也许人家写的东西和你没关系。总之只要检测到有其他线程进来了就算失败)，回滚此前已做的操作，然后从头再来。

无障碍在冲突激烈时会导致临界区中的所有线程都不停的在回滚，极大的影响了程序的性能。严重时，没有一个线程能走出这个回滚的泥潭，形成一种伪死锁的状态。

**无锁(Lock-Free)**

无锁是一种特殊的无障碍。其诞生的目的就是为了解决无障碍"没有一个线程能走出这个回滚的泥潭"的问题。也就是说在任意时刻，无锁都能保证临界区中的线程中至少有一个可以在有限的步骤内离开临界区。因为仅能保证1个，因此排在后面的线程可能会面临饥饿的问题。

**无等待(Wait-Free)**

无等待在无锁的基础上又做了进一步的优化，其解决了无锁中的"饥饿"问题。无锁仅保证同一时刻内只有一个线程能在有限步骤内退出临界区，而无等待更进一步，要求临界区中的所有线程都能在有限的步骤内退出临界区。

无等待所采取的一种典型的"解决策略"就是RCU(Read-Copy-Update)：想要修改临界区的数据时，不能直接在临界区上修改，而是copy出一份当前的副本，而后在这个副本上修改，随后再在合适的时机写回。这样保证临界区不是"脏"的，也不会是写线程写到一半的中间态。因此读线程在进入临界区时就不需要任何并发控制了。这个做法与Git等代码版本控制库的做法很类似，大家可以对比着理解。