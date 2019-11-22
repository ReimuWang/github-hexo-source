---
title: Java 并发-读写锁ReadWriteLock
date: 2018-01-08 15:34:36
tags: [Java,并发,ReadWriteLock,读写锁]
categories: Java 并发
---

java.util.concurrent.locks.ReadWriteLock是JDK1.5起提供的读写分离锁。其目的是为了解决操作系统中的读者写者问题:

```
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable

public interface ReadWriteLock
```

需要注意的是，它并没有实现Lock接口。

<!-- more -->

在读者写者的应用场景中，对于普通的ReentrantLock而言，访问临界区的线程只有一种身份，并不会区分读者与写者：这就导致任意两个访问临界区的线程都是冲突对立的。然而实际上，读者并不会修改临界区，因此读者与读者之间不应该阻塞。ReadWriteLock就是为了优化这种场景而设计的。

这种优化在读远多于写时对性能的提升非常明显，我们不妨来看下面的例子：

```
import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Test {

    private static Lock LOCK = new ReentrantLock();

    private static int VALUE;

    public static int read() {
        int result = 0;
        Test.LOCK.lock();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        result = Test.VALUE;
        Test.LOCK.unlock();
        return result;
    }

    public static void write(int v) {
        Test.LOCK.lock();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Test.VALUE = v;
        Test.LOCK.unlock();
    }

    public static void main(String[] args) {
        long begin = System.currentTimeMillis();
        Runnable read = new Runnable() {
            @Override
            public void run() {
                Test.read();
            }
        };
        Runnable write = new Runnable() {
            @Override
            public void run() {
                Test.write(new Random().nextInt());
            }
        };
        int readCount = 18;
        int writeCount = 2;
        ExecutorService esRead = Executors.newFixedThreadPool(readCount);
        ExecutorService esWrite = Executors.newFixedThreadPool(writeCount);
        for (int i = 0; i < readCount; i++) esRead.submit(read);
        for (int i = 0; i < writeCount; i++) esWrite.submit(write);
        esRead.shutdown();
        esWrite.shutdown();
        try {  
            boolean loop = true;  
            do {  
                loop = (!esRead.awaitTermination(10, TimeUnit.MILLISECONDS)) ||
                       (!esWrite.awaitTermination(10, TimeUnit.MILLISECONDS)) 
                        ;
            } while(loop);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println((System.currentTimeMillis() - begin) / 1000);
    }
}
```

输出:

```
20
```

此时使用的是普通的ReentrantLock，耗时不多不少正好20秒，符合预期。现在我们使用ReadWriteLock：

```
import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class Test {

    private static ReadWriteLock LOCK = new ReentrantReadWriteLock();

    private static Lock READ_LOCK = Test.LOCK.readLock();

    private static Lock WRITE_LOCK = Test.LOCK.writeLock();

    private static int VALUE;

    public static int read() {
        int result = 0;
        Test.READ_LOCK.lock();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        result = Test.VALUE;
        Test.READ_LOCK.unlock();
        return result;
    }

    public static void write(int v) {
        Test.WRITE_LOCK.lock();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Test.VALUE = v;
        Test.WRITE_LOCK.unlock();
    }

    public static void main(String[] args) {
        long begin = System.currentTimeMillis();
        Runnable read = new Runnable() {
            @Override
            public void run() {
                Test.read();
            }
        };
        Runnable write = new Runnable() {
            @Override
            public void run() {
                Test.write(new Random().nextInt());
            }
        };
        int readCount = 18;
        int writeCount = 2;
        ExecutorService esRead = Executors.newFixedThreadPool(readCount);
        ExecutorService esWrite = Executors.newFixedThreadPool(writeCount);
        for (int i = 0; i < readCount; i++) esRead.submit(read);
        for (int i = 0; i < writeCount; i++) esWrite.submit(write);
        esRead.shutdown();
        esWrite.shutdown();
        try {  
            boolean loop = true;  
            do {  
                loop = (!esRead.awaitTermination(10, TimeUnit.MILLISECONDS)) ||
                       (!esWrite.awaitTermination(10, TimeUnit.MILLISECONDS)) 
                        ;
            } while(loop);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println((System.currentTimeMillis() - begin) / 1000);
    }
}
```

经过大量测试，输出为3或4。较之此前的20，性能显然得到了极大的提升。

由该小例子我们也能具体得知，ReadWriteLock作为一个接口，与Lock是一级的，其实现类为ReentrantReadWriteLock。然后再由ReentrantReadWriteLock生成具体的读锁及写锁。因为到读锁及写锁这一层时，锁已没有什么特殊性，因此其实现的是Lock接口。