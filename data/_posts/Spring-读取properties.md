---
title: Spring-读取properties
date: 2018-02-06 22:21:36
tags: [Spring,properties]
categories: Spring
---

本文介绍的方法为通过注解直接将properties文件中的值注入到某个已被bean管理的实例的实例成员变量中。该实例通常为单例。因为如果不是单例而其某实例变量又被注入了相同的值的话，那么该字段就不该声明为实例变量，而应是类变量。

<!-- more -->

首先创建配置文件config.properties，内容如下：

```
cTest1=1
cTest2=reimu
cTest3=2.4
```

properties中可填入任意类型的值(原则上不支持中文字符，如果实在要用请用UTF-8编码，不过不推荐这么做，因为没人能直接看得懂UTF-8编码后的汉字，不利于后期配置文件的维护)。

properties是按照

```
key=value
```

的方式存储的，其中key不能包含.，例如：

```
view.frame.size=50
```

试图存储这样的key是不可以的。

随后在Spring的配置文件applicationContext.xml的beans中添加如下bean：

```
<bean id="config" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
  <property name="locations">  
    <array>
      <value>classpath:config.properties</value>
    </array>
  </property>
</bean>
```

其中array标签中填入配置文件config.properties的路径。既然标签名为数组，那么自然可在其中填入多个properties，本文为了方便举例，只填入一个。

最后，在需注入的实例类中按如下方式声明字段(注意该类必须加载入beans的管理体系中)：

```
import org.springframework.beans.factory.annotation.Value;

public class Test {

    private int test1;

    private String test2;

    private double test3;

    @Value("#{config.cTest1}")
    public void setTest1(int test1) {
        this.test1 = test1;
    }

    @Value("#{config.cTest2}")
    public void setTest2(String test2) {
        this.test2 = test2;
    }

    @Value("#{config.cTest3}")
    public void setTest3(double test3) {
        this.test3 = test3;
    }
}
```

如上代码所示，当然实例变量名是可以与properties对应的key值不同的。