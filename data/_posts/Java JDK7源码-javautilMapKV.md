---
title: Java JDK7源码-java.util.Map&lt;K,V&gt;
date: 2018-11-08 13:48:36
tags: [Java,源码]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public interface Map<K,V> {
    // 查询操作

    /**
     * 返回map中键值对的个数
     * 如果该个数大于Integer.MAX_VALUE，则返回Integer.MAX_VALUE
     */
    int size();

    /**
     * 若map不包含任何键值对则返回true
     */
    boolean isEmpty();

    /**
     * 若map包含key则返回true
     * 更确切的说，当且仅当map包含k，有key==null ? k==null : key.equals(k)时，返回true
     * 
     * @throws ClassCastException key的类型与map不合(可选)
     * @throws NullPointerException key==null且map禁止包含null(可选)
     */
    boolean containsKey(Object key);

    /**
     * 若map中存在1个或多个key的值为入参则返回true
     * 更确切的说，当且仅当map包含至少一个值v，有value==null ? v==null : value.equals(v)时，返回true
     * 通常来说，在多数实现中，本操作的时间复杂度为线性阶(依map的大小)
     * 
     * @throws ClassCastException value的类型与map不合(可选)
     * @throws NullPointerException value==null且map禁止包含null(可选)
     */
    boolean containsValue(Object value);

    /**
     * 返回map中key所对应的值
     * 若map中未包含key，则返回null
     *
     * 更确切的说，若map包含键值对k-v，并有
     * key==null ? k==null : key.equals(k)
     * 则返回v
     * 反之返回null
     * 因为map中键不允许重复，则至多只会有一个对应的值，因此不会产生歧义
     *
     * 基于map中的键值是否允许为null，共有如下四种情况：
     * 
     * 1. 键，值均不允许为null
     * 返回null: map中不包含key
     * 返回非null: map中包含key
     * 
     * 2. 键允许为null，值不允许为null
     * 返回null: map中不包含key
     * 返回非null: map中包含key
     * 
     * 3. 键不允许为null，值允许为null
     * 返回null: 无法判断。有可能map有不包含key，也有可能map中包含key，不过其值为null
     * 返回非null: map中包含key
     * 
     * 4. 键值均允许为null
     * 返回null: 无法判断。有可能map有不包含key，也有可能map中包含key，不过其值为null
     * 返回非null: map中包含key
     * 
     * 由此分析，是否能根据本方法的返回值判断map中是否包含key，与键是否允许为null无关，只与值是否允许为null有关
     * 在无法判断时，可调用containsKey(Object key)辨认
     *
     * @throws ClassCastException key的类型与map不合(可选)
     * @throws NullPointerException key==null且map禁止包含null(可选)
     */
    V get(Object key);

    // 改变操作

    /**
     * 本方法属于破坏性方法，可选
     * 将key-value作为键值对存入map
     * 若map此前包含key，即当且仅当containsKey(key)返回true，则其原值将被新的value替换，并返回原值
     * 若map此前未包含key，则返回null
     * 需要注意的是，返回null并不能说明map此前未包含key，也有可能map此前包含key，只不过该key的原值就为null(当前，前提是实现类允许值为null)
     *
     * @throws UnsupportedOperationException map不支持本方法
     * @throws ClassCastException key或value因其类型禁止被插入map
     * @throws NullPointerException key为null且map的键不允许为null 或 value为null且map的值不允许为null
     * @throws IllegalArgumentException key或value因其某些属性禁止被插入map
     */
    V put(K key, V value);

    /**
     * 本方法属于破坏性方法，可选
     * 若map中包含key，则将其从map中移除
     * 更确切的说，若map包含键值对k-v，并有
     * key==null ?  k==null : key.equals(k)
     * 则移除该键值对
     * 因为map中键不允许重复，则至多只会移除一个键值对，因此不会产生歧义
     * 
     * 若有键值对被移除，则本方法返回对应的值。反之返回null
     * 需要注意的是，返回null并不能说明没有键值对被移除，也有可能map此前包含key，只不过该key的值就为null(当前，前提是实现类允许值为null)
     *
     * 一旦本方法被调用，map将不再包含key
     *
     * @throws UnsupportedOperationException map不支持本方法
     * @throws ClassCastException key的类型与map不合(可选)
     * @throws NullPointerException key为null且map的键不允许为null(可选)
     */
    V remove(Object key);


    // 批量操作

    /**
     * 本方法属于破坏性方法，可选
     * 将m中所有的键值对复制入map
     * 本方法等效于遍历m，随后将m中所有的键值对以put(K key, V value)依次插入map中
     * 本接口并未约束如下情况时的解决策略：在键值对存入map的过程中m发生变化
     * 
     * @throws UnsupportedOperationException map不支持本方法
     * @throws ClassCastException m中的某个key或value因其类型禁止被插入map
     * @throws NullPointerException m==null 或m中存在为null的key且map的键不允许为null 或 m中存在为null的value且map的值不允许为null
     * @throws IllegalArgumentException m中的某个key或value因其某些属性禁止被插入map
     */
    void putAll(Map<? extends K, ? extends V> m);

    /**
     * 本方法属于破坏性方法，可选
     * 移除map中所有的键值对
     * 本方法调用后map将为空
     * 
     * @throws UnsupportedOperationException map不支持本方法
     */
    void clear();


    // 视图

    /**
     * 返回一个视图，该视图的类型为Set，该set由map中所有的key组成
     * 既然该set是视图，那么作用于map之上的改变就会反映在该set上，反之亦然
     * 
     * 若set正在迭代的过程中，map因非set的原因发生了结构性变化
     * (也就是不是由set的Iterator导致的变化，或者更具体的说，不是由set的Iterator的remove()方法导致的变化)
     * 则迭代结果将被置为未定义
     * 
     * 通过set的Iterator的Iterator.remove
     * 或set本身的Set.remove,removeAll,retainAll,clear
     * 操作，可以将key自set中移除，同时，map中的以对应key为键的键值对也会被移除
     * 
     * 该set视图不支持添加操作，即不支持add,addAll
     * (当然啦，只添加一个key是无法形成键值对的)
     * 强行调用会抛出UnsupportedOperationException
     * 
     * 注1:keySet()测试
     */
    Set<K> keySet();

    /**
     * 返回一个视图，该视图的类型为Collection，该collection由map中所有的value组成
     * 既然该collection是视图，那么作用于map之上的改变就会反映在该collection上，反之亦然
     * 
     * 若collection正在迭代的过程中，map因非collection的原因发生了结构性变化
     * (也就是不是由collection的Iterator导致的变化，或者更具体的说，不是由collection的Iterator的remove()方法导致的变化)
     * 则迭代结果将被置为未定义
     * 
     * 通过collection的Iterator的Iterator.remove
     * 或collection本身的Collection.remove,removeAll,retainAll,clear
     * 操作，可以将value自collection中移除，同时，map中的以对应value为值的键值对也会被移除
     * 
     * 该collection视图不支持添加操作，即不支持add,addAll
     * (当然啦，只添加一个value是无法形成键值对的)
     * 强行调用会抛出UnsupportedOperationException
     * 
     * 注2:values()测试
     */
    Collection<V> values();

    /**
     * 返回一个视图，该视图的类型为Set，该set由map中所有的键值对组成
     * 既然该set是视图，那么作用于map之上的改变就会反映在该set上，反之亦然
     * 
     * 若set正在迭代的过程中，map因非set的原因发生了结构性变化
     * 则迭代结果将被置为未定义
     * 
     * 通过set的Iterator的Iterator.remove
     * 或set本身的Set.remove,removeAll,retainAll,clear
     * 操作，可以将键值对自set中移除，同时，map中对应的键值对也会被移除
     * 
     * 该set视图不支持添加操作，即不支持add,addAll
     * 强行调用会抛出UnsupportedOperationException
     * 
     * 注3:entrySet()测试
     */
    Set<Map.Entry<K, V>> entrySet();

    /**
     * 一个Entry对象实际上就是map中的一个键值对
     * 
     * Map.entrySet方法会返回一个类型为Set的视图，它的元素就是Entry类型
     * 如果想要获得map中的Entry，唯一的方式就是调用Map.entrySet方法，然后使用iterator迭代该set
     * 迭代出的Map.Entry对象仅在迭代期间有效
     * 更正式的说，除了因为该set本身导致map发生的变化，在通过iterator迭代该set得到Map.Entry对象期间，如果map发生变化，迭代出的Map.Entry对象将被置为未定义
     */
    interface Entry<K,V> {
        /**
         * 返回本entry中存储的key
         *
         * @throws IllegalStateException 如果entry已被自map中移除，那么实现类可以(但不是必须)抛出本异常
         */
        K getKey();

        /**
         * 返回本entry中存储的value
         * 若entry已被自map中移除(通过iterator的remove操作)，结果将被置为未定义
         * 
         * @throws IllegalStateException 如果entry已被自map中移除，那么实现类可以(但不是必须)抛出本异常
         */
        V getValue();

        /**
         * 本方法可选
         * 将entry中存储的值替换为value，而后返回旧值
         * (相应的，map中对应的键值对中的值也会发生变化)
         * 
         * 未定义如下情况时本方法的行为：entry已被自map中移除(通过iterator的remove操作)
         * 
         * @throws UnsupportedOperationException map不支持put操作
         * @throws ClassCastException value因其类型禁止被插入map
         * @throws NullPointerException value==null且map的值不允许为null
         * @throws IllegalArgumentException value因其某些属性禁止被插入map
         * @throws IllegalStateException 如果entry已被自map中移除，那么实现类可以(但不是必须)抛出本异常
         */
        V setValue(V value);

        /**
         * 比较o与entry的相等性
         * 若o同样是Map Entry且o与entry代表的键值对相等，则返回true
         * 更正式的说，如果满足如下条件，则可认为两个entry e1 e2代表的键值对相等：
         * 
         * if (
         * e1.getKey()==null ? e2.getKey()==null : e1.getKey().equals(e2.getKey())
         * ) && (
         * e1.getValue()==null ? e2.getValue()==null : e1.getValue().equals(e2.getValue())
         * )
         * 
         * 以这种方式设计的话，即便e1 e2的实现类不同，也可以正确的判断二者是否相等
         */
        boolean equals(Object o);

        /**
         * 返回entry的hash code值
         * 定义方式如下：
         * (
         * e.getKey()==null ? 0 : e.getKey().hashCode()
         * ) ^ (
         * e.getValue()==null ? 0 : e.getValue().hashCode()
         * )
         * 
         * 这样的定义方式确保了对于任意Entry而言，只要有
         * e1.equals(e2)
         * 即有
         * e1.hashCode()==e2.hashCode()
         * 遵循equals方法与hashCode方法的设计规范
         */
        int hashCode();
    }

    // 比较与哈希

    /**
     * 比较o与map的相等性
     * 若o同样是一个Map且o与map中存储的键值对均相等，则返回true
     * 更正式的说，如果满足如下条件，则可认为两个Map m1 m2相等：
     * m1.entrySet().equals(m2.entrySet())
     * 以这种方式设计的话，即便m1 m2的实现类不同，也可以正确的判断二者是否相等
     */
    boolean equals(Object o);

    /**
     * 返回map的hash code值
     * 定义方式为：
     * map.entrySet()返回的set中的Entry的hash code之和
     * 
     * 这样的定义方式确保了对于任意Map而言，只要有
     * m1.equals(m2)
     * 即有
     * m1.hashCode()==m2.hashCode()
     * 遵循equals方法与hashCode方法的设计规范
     */
    int hashCode();

}
```

# 已整理层级关系

***直接实现本接口的类***

- [java.util.AbstractMap&lt;K,V&gt;](/2018/11/09/Java JDK7源码-javautilAbstractMapKV/)
- [java.util.HashMap&lt;K,V&gt;](/2018/11/09/Java JDK7源码-javautilHashMapKV/)

# 综述

本接口是Java集合框架中的一员。其中K，V分别是map中key与value的类型。

本接口约束了一个由复数个键值对(key-value)构成的map对象。map不能包含重复的key，每个key至多映射至一个value。

本接口(诞生于JDK1.2)取代了抽象类[java.util.Dictionary&lt;K,V&gt;](/2018/11/08/Java JDK7源码-javautilDictionaryKV/)(诞生于JDK1.0)的地位。

本接口提供了3种用于查看map中的内容的集合视图(collection view)：

1. 由map中的key构成的set
2. 由map中的value构成的collection
3. 由map中的键值对(key-value)构成的set

所谓map中元素的顺序，其实就是在使用这3种方式查看map时，对应的collection的iterator返回元素的顺序。某些本接口的实现类，例如TreeMap，会明确保证map中的元素有序；而对于另一些实现而言，例如HashMap，则不会保证这一点。

需要注意的事，如果作为map key的对象时常会变化，并且这种变化在一定程度上会影响到相等性的判断，那么操作该map后得到的结果就无法完全被控制。基于这个原因，map禁止以其自身作为自己的key。同时，虽然并未禁止，但也应极其谨慎对待的是map以其自身作为自己的value：因为此时map的equals及hashCode方法将难以编写。

通常来说，所有的本接口的实现类都需要提供两个"标准的"构造函数(之所以打上引号，是因为接口是无法约束实现类的构造函数的编写规范的，更遑论标准，这只是一个约定俗成的建议，起码JDK中本接口的实现类都是遵循的)：

1. 创建一个空的map的无参构造函数
2. 接收一个Map类型参数的构造函数，它会以入参为基础，创建一个类型为自身实现类的，由相同键值对(key-value)构成的新map

事实上，我们可以使用第二个构造函数完成map的复制：只要入参和待生成的新map从属于相同的实现类型即可。

本接口包含所谓的"破坏性方法"，这些方法会改变它们所操作的map。如果某实现类不打算实现某个破坏性方法，那么对于这个方法，该实现类应抛出UnsupportedOperationException。该约束其实是比较灵活的：即便某实现类不支持某破坏性方法，但如果该次调用不可能产生实际上的破坏性影响，那么是否抛出UnsupportedOperationException就不加限制了，完全由实现类的编写者决定。例如有不可变的map对象，它显然是不支持本接口所约束的putAll(Map&lt;? extends K, ? extends V&gt; m)，不过如果作为入参的m为空，那么本次调用将无论如何也不会修改该不可变map对象，此时不抛出异常也是可以的。

某些本接口的实现类可能会对它所包含的key或value有所限制。例如，某些实现类禁止包含值为null的key或value，某些对其key的类型有所限制。试图插入一个不合规的key或value会抛出unchecked exception(对于那些并非强烈需要实现类抛出的异常，本接口会将其标注为可选)，典型的诸如NullPointerException或ClassCastException。试图查询一个不合规的key或value可能会导致抛出异常，或者仅仅只是返回false。

本接口的很多方法都是基于Object类的equals(Object obj)定义的。例如containsKey(Object key)的文档中就有这样的叙述：

```
当且仅当map包含k，有key==null ? k==null : key.equals(k)时，返回true
```

该叙述并不意味着如果我们以非null的key去调用containsKey(Object key)，就一定需要使用key.equals(k)来比较。只要实现了与equals等效的比较结果，实现类可以基于自身特点编写更优化的解决方案。仍以本方法为例，我们可以举出一个比较通用的优化方法，实现类可以先用hashCode()比较key与k的hashCode是否相同(当然了，这需要key与k所属的类遵循hashCode()与equals()的规范)，这样就可以快速过滤掉hashCode不相等的情况。

# 注1:keySet()测试

```
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class Test {

    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();
        map.put("a", "A");
        map.put("b", "B");
        Set<String> keySet1 = map.keySet();
        Set<String> keySet2 = map.keySet();
        System.out.println(keySet1 == keySet2);
    }
}
```

输出：

```
true
```

说明多次调用keySet返回的是唯一的视图对象。

---

```
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class Test {

    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();
        map.put("a", "A");
        map.put("b", "B");
        Set<String> keySet = map.keySet();
        for (String key : keySet) System.out.print(key + " ");
        System.out.println();
        map.put("c", "C");
        for (String key : keySet) System.out.print(key + " ");
    }
}
```

输出：

```
b a 
b c a 
```

说明作用于map上key的修改可以影响keySet。反之亦然，在此就不写测试代码了。

---

```
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class Test {

    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();
        map.put("a", "A");
        map.put("b", "B");
        Set<String> keySet = map.keySet();
        keySet.add("c");
        for (String key : keySet) System.out.print(key + " ");
    }
}
```

输出：

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractCollection.add(AbstractCollection.java:260)
	at com.ansjseg.Test.main(Test.java:14)
```

即强行调用keySet的add会抛出UnsupportedOperationException。

# 注2:values()测试

```
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;

public class Test {

    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();
        map.put("a", "A");
        map.put("b", "B");
        map.put("c", "B");
        Collection<String> values = map.values();
        values.remove("B");
        for (Entry<String, String> entry : map.entrySet()) System.out.println(entry.getKey() + " - " + entry.getValue());
    }
}
```

输出：

```
c - B
a - A
```

因为map中key是唯一的，因此如果我们移除keySet()返回的视图中的key，那么可以确切的知道哪个键值对遭到了删除。

然而map中的value却是可重复的，因此如果我们如果只移除values()返回的视图中的某个value，在有复数个键值对的value等于被删除值时，被删除的键值对是不可预测的。

---

```
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

public class Test {

    public static void main(String[] args) {
        Map<String, String> map = new HashMap<String, String>();
        map.put("a", "A");
        map.put("b", "B");
        Collection<String> values = map.values();
        values.add("C");
    }
}
```

输出：

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractCollection.add(AbstractCollection.java:260)
	at com.ansjseg.Test.main(Test.java:14)
```

即强行调用values的add会抛出UnsupportedOperationException。

# 注3:entrySet()测试

```
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class Test {

    public static void main(String[] args) {
        Map<String, String> map1 = new HashMap<String, String>();
        map1.put("a", "A");
        map1.put("b", "B");
        Map<String, String> map2 = new HashMap<String, String>();
        map2.put("c", "C");
        Set<Map.Entry<String, String>> entrySet1 = map1.entrySet();
        Set<Map.Entry<String, String>> entrySet2 = map2.entrySet();
        entrySet1.addAll(entrySet2);
    }
}
```

输出：

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractCollection.add(AbstractCollection.java:260)
	at java.util.AbstractCollection.addAll(AbstractCollection.java:342)
	at com.ansjseg.Test.main(Test.java:17)
```

即强行调用entrySet的addAll会抛出UnsupportedOperationException。