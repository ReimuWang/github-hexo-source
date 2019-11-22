---
title: Java 并发-守护线程
date: 2017-10-07 12:14:36
tags: [Java,守护线程]
categories: Java 并发
---

守护线程(Daemon)：完成系统性服务。例如垃圾回收线程，JIT线程。

用户线程：完成应用程序的业务操作。

当用户线程全部结束时，也就意味着程序员指定的需求都已完成了。守护线程要守护的对象已经不存在了，守护线程也会结束。整个应用程序也就自然结束了。

<!-- more -->

小例子：

```
public class Test {

    public static void main(String[] args) {
        Thread daemon = new Thread() {
          @Override
          public void run() {
              while (true) ;
          }
        };
        // 手动将自定义的线程设置为守护线程。
        daemon.setDaemon(true);
        daemon.start();
    }
}
```

执行后，程序会随着main线程的结束而结束。

注意守护线程的设置必须在其启动之前，如果按如下方式改写代码：

```
public class Test {

    public static void main(String[] args) {
        Thread daemon = new Thread() {
          @Override
          public void run() {
              while (true) ;
          }
        };
        daemon.start();
        // 手动将自定义的线程设置为守护线程。
        daemon.setDaemon(true);
        System.out.println("后续");
    }
}
```

运行后程序抛出运行时异常：

```
Exception in thread "main" java.lang.IllegalThreadStateException
	at java.lang.Thread.setDaemon(Thread.java:1388)
	at com.test.Test.main(Test.java:14)

```

此时程序无法随着main线程的结束而结束。换言之daemon.setDaemon(true);失败了。daemon并非作为守护线程而是普通的用户线程在运行着。

main线程因抛出异常而终止，因此其后续的语句无法执行；daemon线程作为普通应用线程不受影响，因此程序会无限循环下去。