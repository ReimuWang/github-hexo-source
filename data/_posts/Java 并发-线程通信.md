---
title: Java 并发-线程通信
date: 2017-07-13 18:33:36
tags: [Java,并发]
categories: Java 并发
---

# 自定义通信对象

<!-- more -->

```
public class Test {

    private static final Signal SIGNAL = new Signal();

    /**
     * 写入线程：只写入一次。
     * 读取线程：直到有数据前等待，只读取一次。
     */
    public static void main(String[] args) {
        Runnable write = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(2000L);    // 模拟写入耗时
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("写入数据完成");
                Test.SIGNAL.setHaveData(true);
            }
        };
        Runnable read = new Runnable() {
            @Override
            public void run() {
                while (!Test.SIGNAL.haveData) {    // 忙等待
                    try {
                        System.out.println("数据未准备好");
                        Thread.sleep(1000L);    // 防止空转次数过多
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("数据已准备好");
            }
        };
        new Thread(write).start();
        new Thread(read).start();
    }
}

final class Signal {

    boolean haveData = false;

    synchronized void setHaveData(boolean haveData) {
        this.haveData = haveData;
    }
}
```

输出：

```
数据未准备好
数据未准备好
写入数据完成
数据已准备好
```

# wait(),notify()和notifyAll()

详见[Java 并发-synchronized](/2017/07/13/Java 并发-synchronized/)。