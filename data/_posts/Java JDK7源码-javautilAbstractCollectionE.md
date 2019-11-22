---
title: Java JDK7源码-java.util.AbstractCollection&lt;E&gt;
date: 2017-06-19 15:55:27
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public abstract class AbstractCollection<E> implements Collection<E> {

    protected AbstractCollection() {
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 返回一个集合的迭代器，该迭代器默认从头开始迭代。
     */
    public abstract Iterator<E> iterator();

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 返回集合包含的元素个数。
     * 若集合包含的元素个数大于Integer.MAX_VALUE，则返回Integer.MAX_VALUE。
     */
    public abstract int size();

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 若集合不包含元素则返回true。反之返回false。
     */
    public boolean isEmpty() {
        return size() == 0;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 若集合包含o则返回true，反之返回false。
     * 更具体的说，当且仅当集合包含至少一个元素e，满足(o==null ? e==null : o.equals(e))时，返回true，反之返回false。
     * 本方法会用迭代器迭代集合中的每个元素，检查每个元素是否与o相等。
     * 
     * 本方法与后文介绍的containsAll(Collection<?> c)方法很相似，可对比学习。
     * 
     * @throws ClassCastException o的类型与集合不相容。
     * @throws NullPointerException o==null且集合不允许空元素存在。
     */
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 本方法是连接集合与数组之间的桥梁。
     * 
     * 返回一个包含集合所有元素的数组，该数组由集合的迭代器遍历而得，数组中元素的顺序与迭代器遍历而得的顺序相同。
     * 返回的数组的长度等于迭代器遍历而得的元素的个数。
     * 在本方法的开始会调用一次size()得到集合的长度
     * 通常情况下这与迭代器遍历而得的元素个数相同。
     * 但是集合在迭代过程中发生结构性变化时，最终返回的数组的长度以迭代器实际取出的元素个数为准，这个长度可能会与之前调用的size()方法时得到的结果不同。
     * 
     * 本方法等价于：
     * List<E> list = new ArrayList<E>(size());
     * for (E e : this) list.add(e);
     * return list.toArray();
     * 
     * 返回的数组与集合之间不存在引用关系(即使集合是基于数组的)，数组中的元素是集合中的元素的浅拷贝。
     * 
     * 本方法与后文介绍的toArray(T[] a)方法很相似，可对比学习。
     */
    public Object[] toArray() {
        Object[] r = new Object[size()];
        Iterator<E> it = iterator();
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext())    // 迭代器得到的元素少于预期，发生截断
                // 注1：Arrays.copyOf(r, i)
                return Arrays.copyOf(r, i);
            r[i] = it.next();
        }
        // 若it.hasNext()==true说明迭代器得到的元素多于预期：即集合在迭代过程中长度增加
        // 注2：finishToArray(r, it)
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 查询操作。
     * 本方法是连接集合与数组之间的桥梁。
     * 
     * 返回包含集合所有元素的数组。返回数组的类型即为a的类型。顺序为迭代器的迭代顺序。
     * 具体规则的伪代码为：
     * if (a.length < collection.size())
     *     不修改a，而是以a的类型新建长度为collection.size()的数组并填入集合中的值，随后返回这个新生成的数组。
     * else if (a.length == collection.size()
     *     将返回的结果直接填入a后返回a(若a中已有值，则a中的原值会被覆盖)。
     * else
     *     数组索引在[0, collection.size() -1]的元素会被集合对应位置的元素覆盖。
     *     数组索引 == collection.size()的元素会被置为null
     *     数组中后续元素(如果有的话)不变。
     * 
     * 若集合不允许包含空元素，则本方法此时可用来计算集合的长度：返回的数组第一次出现null的索引即为集合的长度。
     * 
     * 返回的数组与集合之间不存在引用关系(即使集合的底层就是基于数组实现的)，数组中的元素是集合中元素的浅拷贝。
     * 
     * 小例子：
     * String[] y = x.toArray(new String[0]);
     * x是一个元素类型为String的集合，则上述语句会将x中的元素依序复制一份浅拷贝到数组y。
     * 
     * 关于本方法与前文介绍的toArray()方法，需明确：
     * collection.toArray()
     * collection.toArray(new Object[s])
     * 当s < collection.size()时，上述两行代码的效果等价。
     * 
     * 返回数组的有效长度(数组中属于集合中的元素的长度)等于迭代器遍历而得的元素的个数。
     * 本方法开始时会调用一次size()得到集合的长度，通常情况下这与迭代器遍历而得的元素个数相同。
     * 但当集合在迭代过程中发生结构性变化时，最终返回的数组的有效长度以迭代器实际取出的元素个数为准。
     * 这个长度可能会与之前调用size()时得到的结果不同。
     * 
     * 注4：toArray(T[] a)的调用示例
     * 
     * 本方法与前文介绍的toArray()方法很相似，可对比学习。
     * 
     * @throws ArrayStoreException a的类型collection不支持。
     * @throws NullPointerException null==a。
     */
    public <T> T[] toArray(T[] a) {
        int size = size();
        // 以a中元素的类型为基础，根据反射新声明一个长度为size的数组
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) {    // 迭代器得到元素少于预期，可能原因有2:
                                     // 1.迭代过程中集合的元素减少。
                                     // 2.a.length>size
                if (a == r) {    // a.length>=size，此时操作r与操作a等效。
                    r[i] = null;
                 // 后续else都是在处理a.length<size的情况。
                 // 此时r与a是两个完全不同的数组。
                } else if (a.length < i) {    // 此时少于预期的原因无论是1或2，集合依然无法放入a中。
                    // 注1：Arrays.copyOf(r, i)
                    // 截取r并返回截取后的数组
                    return Arrays.copyOf(r, i);
                } else {    // 此时虽然最初有a.length<size，但是因为原因1，a又能装下集合了。
                            // 因此仍然选择将集合存入a。
                    // 注3：System.arraycopy(r, 0, a, 0, i)
                    // 调用此方法时有：r.length=size>a.length>=i
                    // 因此前有值时操作的均为r，故将r中的数据复制索引[0,i-1]至a。
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {    // a.length==i的情况不需要置null。
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // 迭代器得到元素多于预期。
        // 可能的原因为：迭代过程中集合的元素增加。
        // 注2：finishToArray(r, it)
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    /**
     * 通常情况下，允许创建的最大数组长度
     * 
     * 某些JVM会在数组中存储同步关键词
     * 因此在声明容量较大的数组时应注意避免OutOfMemoryError
     * -8是一个相对安全的经验之谈，具体是否出错还要看实际的JVM实现
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 在调用toArray(T[] a)或toArray()时
     * 若迭代器返回的元素个数多于预期，则会调用本方法返回新数组，对原数组进行扩容
     * 并将迭代器中多于预期的元素填入新数组
     */
    private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        int i = r.length;
        while (it.hasNext()) {
            int cap = r.length;
            if (i == cap) {    // 本if中是一次扩容操作：
                               // 当前待插入索引i==r.length时需扩容。
                // 若本句越界，则不会进入后面的if条件，会在调用Arrays.copyOf(r, newCap);时抛出异常。
                int newCap = cap + (cap >> 1) + 1;
                // 满足进入本if的条件很苛刻：
                // newCap > MAX_ARRAY_SIZE && newCap < Integer.MAX_VALUE。只有7个值。
                // 进入本if说明当前的newCap OutOfMemoryError的风险较高。
                if (newCap - MAX_ARRAY_SIZE > 0)
                    // 使用cap重新计算一个相对安全的newCap
                    // hugeCapacity(cap + 1):见紧随本方法之后的下一个方法
                    // cap + 1为此时能允许的最小长度，因为cap已经不够用了
                    newCap = hugeCapacity(cap + 1);
                // 执行扩容操作。扩容后r已指向了一个全新的数组
                r = Arrays.copyOf(r, newCap);    // 注1
            }
            r[i++] = (T)it.next();
        }
        // 若扩容后的数组过大则将多余位置截断
        return (i == r.length) ? r : Arrays.copyOf(r, i);
    }

    /**
     * 当数组规模极大时才需要调用本方法。
     * minCapacity为调用方能接受的最小扩容后的数组长度。
     * 
     * 本方法判断minCapacity的值系统是否可接受
     * 若能接受则基于minCapacity返回一个合理的扩容后的长度，若不能接受则抛出OutOfMemoryError。
     */
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0)    // minCapacity本来为一个极大的数，此处minCapacity<0说明其已越界。
            throw new OutOfMemoryError
                ("Required array size too large");
        // 如果可能尽量将大小的上限控制为MAX_ARRAY_SIZE。
        // 如果调用方能接受的容量范围在(MAX_ARRAY_SIZE,Integer.MAX_VALUE]，
        // 此时虽然OutOfMemoryError的风险较大，但系统仍会返回Integer.MAX_VALUE。
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 修改操作
     * 本方法始终抛出UnsupportedOperationException
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException e因其类型禁止被插入集合。
     * @throws NullPointerException e==null且集合禁止包含null。
     * @throws IllegalArgumentException e因其某些属性禁止被插入集合。
     * @throws IllegalStateException 因插入限制，此时e不能被插入集合。
     */
    public boolean add(E e) {
        throw new UnsupportedOperationException();
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 修改操作。
     * 若本方法改变了集合则返回true，反之返回false。
     * 
     * 使用集合的迭代器遍历集合
     * 并用iterator.remove()移除集合符合条件的元素e：
     * (o==null ? e==null : o.equals(e))。
     * 若有多个e符合条件，则本方法只会移除iterator遍历得到的第一个。
     * 
     * 若集合包含o且其迭代器未实现remove()，则抛出UnsupportedOperationException。
     * 
     * 本方法与后文介绍的removeAll(Collection<?> c)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException o的类型与集合不相容。
     * @throws NullPointerException o==null且集合不允许空元素存在。
     */
    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 批量查询操作。
     * 若集合包含c中所有元素则返回true，反之返回false。
     *
     * 本方法与前文介绍的contains(Object o)方法很相似，可对比学习。
     * 
     * @throws ClassCastException c中至少有一个元素的类型与集合不相容。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且集合不允许空元素存在。
     */
    public boolean containsAll(Collection<?> c) {
        for (Object e : c)
            if (!contains(e))
                return false;
        return true;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 将c中所有元素插入集合。
     * 若集合因本方法发生变化则返回true，反之返回false。
     * 
     * 本方法未定义如下事件发生时的解决策略：在将c中的元素添加至集合的过程中c发生变化。
     * 这也意味着如下事件的解决策略同样未定义：将一个非空集合添加至自身。
     * 
     * 在c!=null的前提下，若未重写add(E e)，则本方法将抛出UnsupportedOperationException。
     * 
     * 本方法与前文介绍的add(E e)方法很相似，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException c中任意一个元素因其类型禁止被插入集合。
     * @throws NullPointerException c==null或
     *                              c中至少有一个元素为null且集合不允许空元素存在。
     * @throws IllegalArgumentException c中任意一个元素因其某些属性禁止被插入集合。
     * @throws IllegalStateException 因插入限制，此时并非c中所有元素都能被插入集合。
     */
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 从集合中移除所有与c的交集元素(求差集)。
     * 若集合因本方法发生变化则返回true，反之返回false。
     * 
     * 若迭代器未实现remove()且集合至少有一个元素在c中则抛出UnsupportedOperationException。
     * 
     * 本方法与前文介绍的remove(Object o)方法很相似，可对比学习。
     * 本方法与后文介绍的retainAll(Collection<?> c)方法正相反，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException c中至少有一个元素的类型与集合不相容。
     * @throws NullPointerException c==null或
     *                              集合中至少有一个元素为null且c不允许空元素存在。
     */
    public boolean removeAll(Collection<?> c) {
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 保留集合中与c的交集元素。
     * 若集合因本方法发生变化则返回true，反之返回false。
     * 
     * 若迭代器未实现remove()且集合至少有一个元素不在c中则抛出UnsupportedOperationException。
     * 
     * 本方法与前文介绍的removeAll(Collection<?> c)方法正相反，可对比学习。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     * @throws ClassCastException c中至少有一个元素的类型与集合不相容。
     * @throws NullPointerException c==null或
     *                              集合中至少有一个元素为null且c不允许空元素存在。
     */
    public boolean retainAll(Collection<?> c) {
        boolean modified = false;
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            if (!c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    /**
     * 实现接口:java.util.Collection<E>
     * 
     * 批量修改操作
     * 清空集合。
     * 
     * 若迭代器未实现remove()且集合非空则抛出UnsupportedOperationException。
     * 
     * @throws UnsupportedOperationException 集合不支持本方法。
     */
    public void clear() {
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            it.next();
            it.remove();
        }
    }

    /**
     * 字符串变换
     * 返回集合的字符串表述：
     * [e1, e2, e3]
     * 其中e1,e2,e3为集合按其迭代器顺序排列的元素。
     * 
     * 注5 toString()调用示例
     */
    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            // 防止自包含时无限递归
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }

}
```

# 已整理层级关系

***本类直接实现的接口***

- [java.util.Collection&lt;E&gt;](/2017/05/23/Java JDK7源码-javautilCollectionE/)

***直接继承本类的类***

- [java.util.AbstractList&lt;E&gt;](/2017/06/19/Java JDK7源码-javautilAbstractListE/)
- [java.util.AbstractSet&lt;E&gt;](/2017/06/28/Java JDK7源码-javautilAbstractSetE/)
- [java.util.AbstractQueue&lt;E&gt;](/2017/06/30/Java JDK7源码-javautilAbstractQueueE/)

# 综述

本类是Java集合框架中的一员，提供了Collection接口的基本实现。

如果要实现一个不需要改变的集合，程序员仅仅需要继承本类然后提供iterator()(iterator()返回的iterator必须实现hasNext()及next())及size()的实现。

例如，某个不可变的实现如下(为了填入数据，至少还需要初始化数据的代码，我将其放到了构造函数中)：

```
import java.util.AbstractCollection;
import java.util.Collection;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class UnmoCol<E> extends AbstractCollection<E> {

    private Map<E, Object> map = new HashMap<E, Object>();

    public UnmoCol(Collection<E> c) {
        Iterator<E> cIterator = c.iterator();
        while (cIterator.hasNext()) this.map.put(cIterator.next(), new Object());
    }

    @Override
    public Iterator<E> iterator() {
        return this.map.keySet().iterator();
    }

    @Override
    public int size() {
        return this.map.size();
    }
}
```

调用demo如下：

```
import java.util.Collection;
import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;

public class Main {

    public static void main(String[] args) {
        Set<String> set = new HashSet<String>();
        set.add("reimu");
        set.add("wang");
        Collection<String> collection = new UnmoCol<String>(set);
        System.out.println(collection.size());
        Iterator<String> iterator = collection.iterator();
        while (iterator.hasNext()) System.out.println(iterator.next());
    }
}
```

执行后输出如下：

```
2
reimu
wang
```

如果要实现一个可变的集合，程序员必须重写本类的add(E e)方法(否则会抛出UnsupportedOperationException)。然后iterator()返回的iterator必须实现hasNext(),next(),remove()。

例如，某个实现如下：

```
import java.util.AbstractCollection;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class Mocol<E> extends AbstractCollection<E> {

    private Map<E, Object> map = new HashMap<E, Object>();

    @Override
    public Iterator<E> iterator() {
        return this.map.keySet().iterator();
    }

    @Override
    public int size() {
        return this.map.size();
    }

    @Override
    public boolean add(E e) {
        return this.map.put(e, new Object()) == null;
    }
}
```

调用demo如下：

```
import java.util.Collection;
import java.util.Iterator;

public class Main {

    public static void main(String[] args) {
        Collection<String> collection = new Mocol<String>();
        collection.add("reimu");
        collection.add("wang");
        System.out.println(collection.size());
        Iterator<String> iterator = collection.iterator();
        while (iterator.hasNext()) System.out.println(iterator.next());
    }
}

```

执行本demo输出如下：

```
2
reimu
wang
```

Java语言规范建议程序员在继承本类时实现两个构造函数：void(无参)构造函数。接收一个collection的构造函数。

综上所述，现写本类的测试继承类如下：

```
import java.util.AbstractCollection;
import java.util.Collection;
import java.util.HashMap;
import java.util.Iterator;

public class AbstractCollectionTest<E> extends AbstractCollection<E> {

    private HashMap<E, Object> map = new HashMap<E, Object>();

    public AbstractCollectionTest() {
    }

    public AbstractCollectionTest(Collection<E> c) {
        Iterator<E> cIterator = c.iterator();
        while (cIterator.hasNext()) this.map.put(cIterator.next(), new Object());
    }

    @Override
    public Iterator<E> iterator() {
        return this.map.keySet().iterator();
    }

    @Override
    public int size() {
        return this.map.size();
    }

    @Override
    public boolean add(E e) {
        return map.put(e, new Object()) == null;
    }
}
```

# 注1：Arrays.copyOf(r, i)

以r为基础返回一个长度为i的数组。

- 若i&lt;r.length则r发生截断。实际返回的数组为r中索引为[0,i-1]的元素。
- 若i==r.length，则返回的数组与r相等。
- 若i&gt;r.length，返回的数组中索引为[0, r.length-1]的元素为r中对应位置的元素，索引为[r.length, i-1]的元素以null填充。

无论如何，返回的数组都是新生成的，与r不存在引用关系。返回的数组中的元素是r中元素的浅拷贝。

# 注2：finishToArray(r, it)

finishToArray(r, it)会以r为基础声明一个长度足够的新数组r2，并将it中超出预期仍未取完的元素添加至r2，而后返回r2。

# 注3：System.arraycopy(r, 0, a, 0, i)

参数含义依次为：

1. r: 待复制的源数组。
2. 0: 源数组中开始复制的索引。
3. a: 复制目标数组。
4. 0: 目标数组中接收元素的起始索引。
5. i: 复制的元素个数。

# 注4：toArray(T[] a)的调用示例

示例1：

```
import java.util.Collection;
import java.util.HashSet;
import java.util.Set;

public class Main {

    public static void main(String[] args) {
        Set<String> set = new HashSet<String>();
        set.add("reimu");
        set.add("wang");
        Collection<String> collection = new UnmoCol<String>(set);
        String[] a = {"1", "2", "3", "4"};
        String[] a1 = collection.toArray(a);
        for (String e : a1) System.out.println(e);
    }
}
```

输出结果：

```
reimu
wang
null
4
```

---

示例2

```
package com.collectiontest.main;

import java.util.Collection;
import java.util.HashSet;
import java.util.Set;

public class Main {

    public static void main(String[] args) {
        Set<String> set = new HashSet<String>();
        set.add("reimu");
        set.add("wang");
        Collection<String> collection = new UnmoCol<String>(set);
        String[] a = {};
        String[] a1 = collection.toArray(a);
        for (String e : a1) System.out.println(e);
    }
}
```

输出结果：

```
reimu
wang
```

# 注5 toString()调用示例

**自包含测试**

```
import java.util.Collection;

public class Main {

    public static void main(String[] args) {
        Collection<Collection> collection = new AbstractCollectionTest<Collection>();
        collection.add(collection);
        System.out.println(collection);
    }
}
```

输出：

```
[(this Collection)]
```

**非自包含情况测试**

```
import java.util.Collection;
import java.util.HashSet;
import java.util.Set;

public class Main {

    public static void main(String[] args) {
        Set<String> set = new HashSet<String>();
        set.add("reimu");
        set.add("wang");
        Collection<String> collection = new AbstractCollectionTest<String>(set);
        System.out.println(collection);
    }
}
```

输出：

```
[reimu, wang]
```

# 未整理层级关系

***本类直接继承的类***

- [java.lang.Object]()

***直接继承本类的类***

- [java.util.ArrayDeque&lt;E&gt;]()
- [java.util.concurrent.ConcurrentLinkedDeque&lt;E&gt;]()

***直接继承本类的内部类***

- [静态成员内部类java.lang.ProcessEnvironment.CheckedValues]()
- [实例成员内部类java.util.HashMap&lt;K,V&gt;.Values]()
- [实例成员内部类java.util.IdentityHashMap&lt;K,V&gt;.Values]()
- [实例成员内部类java.util.WeakHashMap&lt;K,V&gt;.Values]()
- [实例成员内部类java.util.TreeMap&lt;K,V&gt;.Values]()
- [实例成员内部类java.util.Hashtable&lt;K,V&gt;.ValueCollection]()
- [实例成员内部类java.util.EnumMap&lt;K extends Enum&lt;K&gt;, V&gt;.Values]()
- [实例成员内部类java.util.concurrent.ConcurrentHashMap&lt;K, V&gt;.Values]()
- [实例成员内部类java.util.concurrent.ConcurrentSkipListMap&lt;K,V&gt;.Values&lt;E&gt;]()