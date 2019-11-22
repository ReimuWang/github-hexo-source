---
title: Java 并发-倒计数器CountDownLatch
date: 2018-01-08 16:51:36
tags: [Java,并发,CountDownLatch]
categories: Java 并发
---

CountDownLatch直译为倒计数门栓。其中门栓的含义很直观，仅取其字面意思即可：在计数结束前，将线程用门栓关在门里，待计数结束，再将门栓取下，放线程出来。

有人会将CountDownLatch译为倒计时器。我并不推荐这种翻译，因为这会让使用者产生一种误解：CountDownLatch会等待一段时间。而事实上，CountDownLatch进行的是数字上的倒数，故翻译为倒计数器更为合理。

<!-- more -->

java.util.concurrent.CountDownLatch的类定义为

```
public class CountDownLatch
```

其常用的构造函数为：

```
/**
 * @param count int, 倒计数的个数
 */
public CountDownLatch(int count)
```

我们不妨构造一个小场景：一个工作在开始前需要先完成准备工作，而准备工作由5位不同的工人完成。代码如下：

```
import java.util.Random;
import java.util.concurrent.CountDownLatch;

public class Test {

    private static int WORKER_COUNT = 5;

    private static CountDownLatch CDL = new CountDownLatch(Test.WORKER_COUNT);

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                long begin = System.currentTimeMillis();
                try {
                    Thread.sleep(new Random().nextInt(10) * 1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "准备完成，耗时" + ((System.currentTimeMillis()- begin) / 1000) + "秒");
                Test.CDL.countDown();
            }
        };
        for (int i = 0; i < Test.WORKER_COUNT; i++) new Thread(r, "工人" + i).start();
        Test.CDL.await();
        System.out.println("准备完成，工作开始...");
    }
}
```

输出：

```
工人2准备完成，耗时2秒
工人1准备完成，耗时6秒
工人0准备完成，耗时8秒
工人4准备完成，耗时9秒
工人3准备完成，耗时9秒
准备完成，工作开始...
```