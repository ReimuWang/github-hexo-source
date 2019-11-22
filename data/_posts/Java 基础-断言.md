---
title: Java 基础-断言
date: 2017-09-29 15:30:49
tags: [Java,断言]
categories: Java 基础
---

断言是一种常见的调试方式，很多语言(C,C++,Python,Java)都支持这种机制。因为是调试机制，因此断言默认是不开启的。若要开启断言，则需在启动参数中添加-enableassertions，或简写为-ea。

Java中的断言可有如下两种形式：

<!-- more -->

第一种：

```
assert a > 0;
```

失败时输出：

```
Exception in thread "main" java.lang.AssertionError
	at test.Test.main(Test.java:7)
```

第二种：

```
assert a >= 0 : "a小于0";
```

失败时输出：

```
Exception in thread "main" java.lang.AssertionError: a小于0
	at test.Test.main(Test.java:8)
```

断言失败后，JVM会抛出一个AssertionError错误，它继承自Error。注意，这是一个错误，是不可恢复的。也就表示这是一个严重的问题，开发者必须予以关注并解决之。

断言的设计初衷为辅助调试，因此断言不应影响代码的执行结果。