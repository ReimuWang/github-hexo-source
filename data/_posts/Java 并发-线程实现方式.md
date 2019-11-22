---
title: Java 并发-线程实现方式
date: 2017-10-03 14:28:36
tags: [Java,并发,Thread,Runnable,Callable]
categories: Java 并发
---

JDK1.5以前实现多线程有两种方法：一种是继承Thread类；另一种是实现Runnable接口。两种方式都要通过重写run()方法来定义线程的行为，推荐使用后者，因为Java中的继承是单继承，一个类只有一个父类，如果继承了Thread类就无法再继承其他类了，显然使用Runnable接口更为灵活。

JDK1.5以后创建线程还有第三种方式：实现Callable接口，该接口中的call方法可以在线程执行结束时产生一个返回值，代码如下所示：

<!-- more -->

```
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class Test {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        List<Future<Integer>> list = new ArrayList<>();
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for(int i = 0; i < 10; i++) list.add(executorService.submit(new MyTask(100)));
        int sum = 0;
        for(Future<Integer> future : list) sum += future.get();
        System.out.println(sum);
    }
}

class MyTask implements Callable<Integer> {

    private int upperBounds;

    public MyTask(int upperBounds) {
        this.upperBounds = upperBounds;
    }

    @Override
    public Integer call() throws Exception {
        int sum = 0; 
        for(int i = 1; i <= upperBounds; i++) sum += i;
        return sum;
    }
}
```

输出：

```
50500
```