---
title: Java IO-流
date: 2017-10-12 11:16:49
tags: [Java,IO,流]
categories: Java IO
---

# Java中有几种类型的流？

- 字节流，继承于InputStream，OutputStream
- 字符流，继承于Reader，Writer

<!-- more -->

# 文件复制

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class Test {

    public static void fileCopy(String source, String target) throws IOException {
        try (InputStream in = new FileInputStream(source)) {
            try (OutputStream out = new FileOutputStream(target)) {
                byte[] buffer = new byte[4096];
                int bytesToRead;
                while((bytesToRead = in.read(buffer)) != -1) out.write(buffer, 0, bytesToRead);
            }
        }
    }

    public static void fileCopyNIO(String source, String target) throws IOException {
        try (FileInputStream in = new FileInputStream(source)) {
            try (FileOutputStream out = new FileOutputStream(target)) {
                FileChannel inChannel = in.getChannel();
                FileChannel outChannel = out.getChannel();
                ByteBuffer buffer = ByteBuffer.allocate(4096);
                while(inChannel.read(buffer) != -1) {
                    buffer.flip();
                    outChannel.write(buffer);
                    buffer.clear();
                }
            }
        }
    }

    public static void main(String[] args) throws IOException {
        fileCopyNIO("e:\\1.txt", "e:\\2.txt");
    }
}
```

# Scanner接受键盘输入

```
import java.util.Scanner;

public class Test {
    public static void main(String[] args) {
        try (Scanner scanner = new Scanner(System.in)) {
            System.out.println("String Line:" + scanner.nextLine());
            System.out.println("String:" + scanner.next());
            System.out.println("Boolean:" + scanner.nextBoolean());
            System.out.println("Byte:" + scanner.nextByte());
            System.out.println("short:" + scanner.nextShort());
            System.out.println("Integer:" + scanner.nextInt());
            System.out.println("Long:" + scanner.nextLong());
            System.out.println("float:" + scanner.nextFloat());
            System.out.println("double:" + scanner.nextDouble());
        }
    }
}
```

输出：

```
line
String Line:line
reimu
String:reimu
true
Boolean:true
12
Byte:12
14
short:14
22
Integer:22
342
Long:342
2.1
float:2.1
4.5
double:4.5
```

线程执行至scanner.nextXXX()时会阻塞(基本数据类型中没有针对char的操作)，等待用户输入。当接收到回车时认为用户输入完成。用户在此期间输入的内容会作为scanner.nextXXX()的返回值返回。当传入字节流对应的数据不符合接收函数的类型规范时会抛出异常。