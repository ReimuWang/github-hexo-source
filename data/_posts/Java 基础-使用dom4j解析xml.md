---
title: Java 基础-使用dom4j解析xml
date: 2018-02-06 22:54:49
tags: [Java,dom4j,xml]
categories: Java 基础
---

首先在pom文件中添加dom4j的Maven地址：

```
<dependency>
  <groupId>dom4j</groupId>
  <artifactId>dom4j</artifactId>
  <version>1.6.1</version>
</dependency>
```

<!-- more -->

随后创建配置文件conf.xml，内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>

<幻想乡 设定者="ZUN" 整数值="3" 浮点数值="2.2">
  <长者>八云紫</长者>
  <少女>博丽灵梦</少女>
  <少女>雾雨魔理沙</少女>
  <少女>十六夜咲夜</少女>
</幻想乡>
```

标签值自然是不推荐用中文的(和代码中不推荐将类名及变量名等定义为中文是一个道理)，本文这么写只是为了强调标签值其实也可以是中文的。

然后基本的前置准备代码为：

```
SAXReader sReader = new SAXReader();
Document document = sReader.read(new File("conf.xml"));
Element root = document.getRootElement();
```

其中document代表配置文件本身，而root则是根节点的元素值。根节点必须是唯一的，也就是说：

```
<?xml version="1.0" encoding="UTF-8"?>

<test1></test1>
<test2></test2>
```

这样的xml文件是非法的。

从宏观上看，Element是xml文件基本的，也是唯一的数据结构。所谓xml文件就是由一个个Element以一定的架构关联在一起的集合。每个Element都由以下4部分构成：

- 标签名

- 属性列表

- 文本值

- 子元素列表

# 标签名

是Element最基本的属性，用来表示元素是什么：

```
System.out.println(root.getName());
```

输出：

```
幻想乡
```

# 属性列表

属性(Attribute)列表出现在标签中，可有多个，用来表示元素有哪些属性，其格式为key-value对。

若示例代码为：

```
List<Attribute> list = root.attributes();
for (Attribute attribute : list) {
    System.out.println(attribute.getName() + " : " + attribute.getValue());
}
```

输出：

```
设定者 : ZUN
整数值 : 3
浮点数值 : 2.2
```

这样便可以列出某元素中的所有属性。需要注意的是xml文件中所有的属性值都必须用双引号引起来，且attribute.getValue()的返回值为String。

如果我们想依元素属性的key值获得特定的属性：

```
Attribute a = root.attribute("设定者");
```

这样便拿到了root节点中的属性"设定者"。

# 文本值

元素的值可以为一个简单的文本，也可以是一系列的子元素。二者只能取其一。

文本值获取举例：

```
Element e = root.element("长者");
System.out.println(e.getText());
```

这样便取到了"幻想乡.长者"中存储的文本值。纯从存储的角度来讲，除了根节点外，这种元素其实都是没有意义的，因为它们完全可以化为其父元素的一个属性。不过从业务逻辑的角度考虑：属性是元素的特征，而子元素是元素的孩子，二者的含义还是有很大的区别的。

# 子元素列表

可依如下方式迭代某元素下所有的子元素：

```
Iterator<Element> iterator = root.elementIterator();
while (iterator.hasNext()) {
    Element e = iterator.next();
    System.out.println(e.getName());
}
```

输出：

```
长者
少女
少女
少女
```

至于按照子元素标签值取某元素下的特定子元素，上一小节中已在取"幻想乡.长者"的例子中展示过。

这里需要注意的是，若取的标签值为"少女"，得到的子元素不止一个的时候该怎么办呢？事实上，此时如果依然采用上一小节中的root.element("长者")的方法返回是空：这其实很好理解，程序并不知道该在复数个结果中选择哪个返回给我们。

解决办法也很简单：

```
Iterator<Element> iterator = root.elementIterator("少女");
while (iterator.hasNext()) {
    Element e = iterator.next();
    System.out.println(e.getName());
}
```

指定标签值即可，输出：

```
少女
少女
少女
```