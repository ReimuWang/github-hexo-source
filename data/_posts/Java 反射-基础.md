---
title: Java 反射-基础
date: 2017-10-13 10:39:49
tags: [Java,反射]
categories: Java 反射
---

[JVM-类加载机制](/2017/11/23/JVM-类加载机制/)的第一阶段加载是整个类加载过程中最为灵活的一个阶段。[JVM-类加载机制](/2017/11/23/JVM-类加载机制/)中具体介绍了在类信息来源上可以玩的花活，其实总结一下，来源主要可分为两大类：

1. 编译期已知。或者具体来说，就是由javac编译器生成的class文件。
2. 编译期未知。会在运行期类加载-加载阶段动态的从某一特定数据源获取class文件。

前者也被称作RTTI(Run-Time Type Identification，即运行时类型识别)，后者则被称作反射。

在Java API中，反射的功能大多集中在包java.lang.reflect中。

# 获得一个类的类对象

- 类型.class，例如：String.class
- 对象.getClass()，例如：str.getClass()
- Class.forName()，例如：Class.forName("java.lang.String")

大多的反射操作基本都是通过这个类对象完成的。

<!-- more -->

# 通过反射创建对象

类对象.newInstance()，例如：String.class.newInstance()。默认调用无参构造函数，因此要求被调用类中必须有显式定义的可调用的无参构造函数。违反该规定可通过编译，但是运行时会抛出异常：

```
public class Test {

    public Test(int i) {
    }

    public static void main(String[] args) throws InstantiationException, IllegalAccessException {
        Test.class.newInstance();
    }
}
```

输出：

```
Exception in thread "main" java.lang.InstantiationException: com.test.Test
	at java.lang.Class.newInstance(Class.java:368)
	at com.test.Test.main(Test.java:9)
```

---

类对象.getConstructor()：返回指定参数类型、具有public访问权限的构造器对象。

类对象.getDeclaredConstructor()：返回指定参数类型、所有声明的(包括private)构造器对象。 

获得构造器对象(Constructor)后调用其newInstance()方法创建对象。例如：

```
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class Test {

    private int i;

    /**
     * 若类型为int则会报错:
     * java.lang.NoSuchMethodException: com.test.Test.<init>(java.lang.Integer)
     */
    public Test(Integer i) {
        this.i = i;
    }

    public static void main(String[] args) throws InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException, NoSuchMethodException, SecurityException {
        Constructor<Test> constructor = Test.class.getConstructor(Integer.class);
        System.out.println(constructor.newInstance(5).i);
    }
}
```

输出：

```
5
```

# 通过反射获取和设置对象私有字段

```
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;

public class Test {

    /**
     * 通过反射获取对象指定字段(变量)的值
     * @param target 目标对象
     * @param fieldName 字段的名字
     * @return 字段的值
     */
    public static Object getValue(Object target, String fieldName) {
        Class<?> clazz = target.getClass();
        String[] fs = fieldName.split("\\.");

        try {
            for(int i = 0; i < fs.length - 1; i++) {
                Field f = clazz.getDeclaredField(fs[i]);
                f.setAccessible(true);
                target = f.get(target);
                clazz = target.getClass();
            }

            Field f = clazz.getDeclaredField(fs[fs.length - 1]);
            f.setAccessible(true);
            return f.get(target);
        }
        catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
 
    /**
     * 通过反射给对象的指定字段赋值
     * @param target 目标对象
     * @param fieldName 字段的名称
     * @param value 值
     */
    public static void setValue(Object target, String fieldName, Object value) {
        Class<?> clazz = target.getClass();
        String[] fs = fieldName.split("\\.");
        try {
            for(int i = 0; i < fs.length - 1; i++) {
                Field f = clazz.getDeclaredField(fs[i]);
                f.setAccessible(true);
                Object val = f.get(target);
                if(val == null) {
                    Constructor<?> c = f.getType().getDeclaredConstructor();
                    c.setAccessible(true);
                    val = c.newInstance();
                    f.set(target, val);
                }
                target = val;
                clazz = target.getClass();
            }

            Field f = clazz.getDeclaredField(fs[fs.length - 1]);
            f.setAccessible(true);
            f.set(target, value);
        }
        catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

# 通过反射调用对象的方法

```
import java.lang.reflect.Method;

public class Test {

    public static void main(String[] args) throws Exception {
        String str = "hello";
        Method m = str.getClass().getMethod("toUpperCase");
        System.out.println(m.invoke(str));    // 输出HELLO
    }
}
```