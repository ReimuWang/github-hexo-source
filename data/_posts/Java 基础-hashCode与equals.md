---
title: Java 基础-hashCode与equals
date: 2017-10-10 15:29:49
tags: [Java,hashCode,equals]
categories: Java 基础
---

# hashCode()与equals(Object obj)的设计原则

在设计hashCode()及equals(Object obj)时，应满足如下两点：

1. hashCode()相等，equals(Object obj)不一定相等。

2. equals(Object obj)相等，hashCode()必然相等。

<!-- more -->

如果违背了第一条，即强行要求hashCode()相等必有equals(Object obj)相等，那么配合第二条，hashCode()与equals(Object obj)就变成一回事了。换句话说，我们为了能够区分两个对象，必须保证能为其分配一个唯一的int型数字，这样做的成本太高了，对于集合型的对象而言，这甚至几乎是不可做到的。

如果违背了第二条，那么基于该规则定义的法则将受到动摇，程序可能发生难以预期的错误。

例如，Map之所以能快速找到作为key的对象，是因为其查找过程分为两步：首先通过hashCode()计算出对象在散列表中的位置，然后在该位置中使用equals(Object obj)进行线性查找。插入过程也类似：首先通过hashCode()计算出对象在散列表中的位置，然后通过equals(Object obj)判断该位置是否已有待插入对象，若已有则拒绝插入。

显然，如果违背了第二条，在插入时equals(Object obj)相等的对象因为不在同一个散列值下都可以插入Map中了，这违背了Map的key不可重复的原则。而查到时，自然也无法正确定位到满足equals(Object obj)相等的对象。例如：

```
import java.util.HashMap;
import java.util.Map;

public class Test {

    private int hashCode;

    public Test(int hashCode) {
        this.hashCode = hashCode;
    }

    @Override
    public boolean equals(Object obj) {
        return true;
    }

    @Override
    public int hashCode() {
        return this.hashCode;
    }

    public static void main(String[] args) {
        Map<Test, Object> map = new HashMap<Test, Object>();
        map.put(new Test(0), null);
        map.put(new Test(1), null);
        System.out.println(map.size());    // 输出2
        System.out.println(map.containsKey(new Test(2)));    // 输出false
    }
}
```

# equals(Object obj)必须满足的原则

- 自反性：x.equals(x)必须返回true。

- 对称性：x.equals(y)返回true时，y.equals(x)也必须返回true。

- 传递性：x.equals(y)和y.equals(z)都返回true时，x.equals(z)也必须返回true。

- 一致性：当x和y引用的对象信息没有被修改时，多次调用x.equals(y)应该得到同样的返回值。而且对于任何非null值的引用x，x.equals(null)必须返回false。

# equals(Object obj)设计技巧

1. 使用==操作符检查参数是否为这个对象的引用。

2. 使用instanceof操作符检查参数是否为正确的类型。

3. 对于类中的关键属性，检查参数传入对象的属性是否与之相匹配。

4. 编写完equals方法后，确认是否满足自反性，对称性，传递性及一致性。

5. 重写equals时总是要重写hashCode。

6. 不要将equals方法参数中的Object对象替换为其他的类型，在重写时不要忘掉@Override注解。


# 为何hashCode()常用素数31？

以java.util.AbstractList中的hashCode()为例：

```
public int hashCode() {
    int hashCode = 1;
    for (E e : this)
        hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
    return hashCode;
}
```

为何是素数：素数减少了相乘后质因数分解的个数，和合数相比减少了冲突的概率。

为何是31：如果仅仅是为了避免冲突，理论上越大的素数越利于避免冲突。但是过大的素数又有导致溢出的风险。31就是这样一个大小适中的素数。同时，31 * num 等价于(num << 5) – num，JVM对这个计算做过特殊的优化。