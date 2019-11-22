---
title: Java 反射-动态代理
date: 2017-12-19 11:00:49
tags: [Java,反射,动态代理]
categories: Java 反射
---

提到字节码生成技术，大家往往都会想到Javassist,CGLib,ASM等操作字节码的类库，给人一种很酷炫的感觉。实际上字节码生成技术距离我们并没那么遥远。在我们接触Java伊始遇到的javac编译器就是一个用Java写成的字节码生成器。此外Web服务器中的JSP编译器，编译时植入的AOP框架，甚至JVM在实现反射时都有可能会在运行时生成字节码以提高执行速度。

<!-- more -->

本文所讨论的反射中的动态代理技术就是字节码生成技术中一个相对简单的应用。

对于很多程序员而言，动态代理并不熟悉，也没接触过java.lang.reflect.Proxy或java.lang.reflect.InvocationHandler。不过我想大部分人都用Spring框架做过Bean的组织管理，而Spring的Bean管理本质上应用的就是动态代理技术。

如果我们将程序员手写代理类的方式称为静态代理的话，那么动态代理指得就是代理类无需程序员编写，将由JVM自动生成。动态代理的优势并不在于节省程序员编写代理类的那一点点的开发成本，而是从本质上提高程序的灵活性：动态代理可以在原始类和接口还未知的时候，就确定代理类的代理行为。这样就实现了代理类与原始类的解耦，从而能让代理类灵活的重用于不同的应用场景中。

静态代理：

```
public class Test {

    public static void consumer(ProxyInterface pi) {
        pi.say();
    }

    public static void main(String[] args) {
        Test.consumer(new ProxyObject());
    }
}

interface ProxyInterface {
    public void say();
}

class RealObject implements ProxyInterface {
    @Override
    public void say() {
        System.out.println("say");
    }
}

class ProxyObject implements ProxyInterface {
    @Override
    public void say() {
        new RealObject().say();
    }
}
```

输出：

```
say
```

---

改造为动态代理：

```
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class Test {

    public static void customer(ProxyInterface pi) {
        pi.say();
    }

    public static void main(String[] args) {
        RealObject real = new RealObject();

        ClassLoader loader = ProxyInterface.class.getClassLoader();
        Class<?>[] interfaces = new Class[]{ProxyInterface.class};
        InvocationHandler h = new ProxyObject(real);
        ProxyInterface proxy = (ProxyInterface)Proxy.newProxyInstance(loader, interfaces, h);
        Test.customer(proxy);
    }
}


interface ProxyInterface {
    void say();
}

class RealObject implements ProxyInterface {
    @Override
    public void say(){
        System.out.println("say");
    }
}

class ProxyObject implements InvocationHandler {

    private Object real = null;

    public ProxyObject(Object real) {
        this.real  = real;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(this.real, args);
    }
}
```

输出：

```
say
```

上述动态代理的代码中最核心的方法就是Proxy.newProxyInstance()了。该方法返回了一个实现ProxyInterface接口，实际业务逻辑类型是RealObject的动态代理类。该代理类程序员并未编写，是在运行期动态生成的。

如果我们跟踪方法Proxy.newProxyInstance()的源码，会发现它进行了验证，优化，缓存，同步，生成字节码，显式类加载等操作。其中最重要的是生成字节码的操作，也是本方法的核心功能点，是通过sun.misc.ProxyGenerator.generateProxyClass()方法完成的。该方法生成的就是动态代理类的字节码，其基本思路并不复杂，基本就是在模仿javac编译器，为接口中定义的每一个方法，以及从java.lang.Object中继承来的equals(),hashCode(),toString()都生成对应的实现，实现的具体的逻辑由程序员写在实现InvocationHandler接口的对象的invoke()方法中。

需要注意的是，使用上述方法实现的动态代理技术是比较原始和粗糙的，实现的动态代理类也是高度模板化的，虽说是动态的，但灵活度并没有那么高。因此实际开发中，还是推荐大家使用大神们已经封装好的各种操作字节码的类库。这二者的关系就好比Javascript与Jquery。