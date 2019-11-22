---
title: Java JDK7源码-java.util.ArrayList&lt;E&gt;
date: 2017-07-06 15:48:14
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 底层数组默认的初始化容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 供本类空实例使用的空数组
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 用于存储arrayList中的元素的数组
     * arrayList的容量即为本数组的长度
     * 
     * 使用
     * new ArrayList<>()声明的arrayList其内部均有：
     * elementData == EMPTY_ELEMENTDATA
     * 在插入第一个元素时elementData会被扩展为DEFAULT_CAPACITY
     */
    private transient Object[] elementData;

    /**
     * arrayList的长度
     * 注意，本字段不是arrayList的容量，而是arrayList包含的元素个数
     */
    private int size;

    /**
     * 以initialCapacity为初始化容量构建一个空arrayList
     * 
     * @throws IllegalArgumentException：initialCapacity < 0。
     */
    public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

    /**
     * 以默认容量10构造一个空arrayList
     * 刚构造完成时数组依然是EMPTY_ELEMENTDATA，即空数组。
     * 需要第一个元素插入进来后才会自动扩展为长度为10的数组
     */
    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray()可能不返回Object[](见6260652)
        // 说明：
        // 因为c可以是任意的Collection实现
        // 而Collection对toArray()的返回值仅仅是要求Object[]
        // 即实际的返回可以是任意Object的子类
        if (elementData.getClass() != Object[].class)
            // 注1:Arrays.copyOf(r, i, newType)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }

    /**
     * 裁剪arrayList的容量为arrayList当前的实际长度
     * 可以使用本方法达到arrayList的存储空间占用最小化
     */
    public void trimToSize() {
        // 注2:modCount
        modCount++;
        if (size < elementData.length) {
            // 注3：Arrays.copyOf(r, i)
            elementData = Arrays.copyOf(elementData, size);
        }
    }

    /**
     * 公开出去的API方法
     * 调用本方法时并不知道当前容量是否能满足minCapacity的需求
     * 本方法会确保当前容量能满足minCapacity的需求(即如果不够用的话则需要扩容)
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != EMPTY_ELEMENTDATA)
            ? 0
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    /**
     * 供ArrayList类内部调用
     * 调用本方法时并不知道当前容量是否能满足minCapacity的需求
     * 本方法会确保当前容量能满足minCapacity的需求(即如果不够用的话则需要扩容)
     */
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    /**
     * 调用本方法时并不知道当前容量是否能满足minCapacity的需求
     * 本方法会确保当前容量能满足minCapacity的需求(即如果不够用的话则需要扩容)
     */
    private void ensureExplicitCapacity(int minCapacity) {
        // 注2:modCount
        modCount++;

        // 内存溢出保护代码
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 数组能申请的最大长度
     * 某些JVM会在数组中存储一些头信息
     * 试图申请过大长度的数组可能会导致OutOfMemoryError：
     * 所需数组长度超过JVM限制
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 本方法被调用说明当前数组的容量确实是不够minCapacity的需求用了
     * 扩充数组容量以满足其至少能容纳minCapacity个元素
     */
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 注3：Arrays.copyOf(r, i)
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0)
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回arrayList的元素个数
     */
    public int size() {
        return size;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 若arrayList不包含元素则返回true
     */
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 若arrayList包含o则返回true
     * 更一般的来说，当且仅当arrayList至少包含一个满足如下条件的元素e时返回true：
     * (o==null ? e==null : o.equals(e))
     */
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回o在arrayList中第一次出现时的索引
     * 若arrayList中不包含o则返回-1
     * 
     * 更一般的来说，返回满足如下条件的i的最小值：
     * (o==null ? get(i)==null : o.equals(get(i)))
     * 若无法找到满足条件的i则返回-1
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回o在arrayList中最后一次出现时的索引
     * 若arrayList中不包含o则返回-1
     * 
     * 更一般的来说，返回满足如下条件的i的最大值：
     * (o==null ? get(i)==null : o.equals(get(i)))
     * 若无法找到满足条件的i则返回-1
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 返回arrayList的一个浅拷贝(其所包含的元素本身并未被复制)
     */
    public Object clone() {
        try {
            @SuppressWarnings("unchecked")
            ArrayList<E> v = (ArrayList<E>) super.clone();
            // 注3：Arrays.copyOf(r, i)
            v.elementData = Arrays.copyOf(elementData, size);
            // # 注2:modCount
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 因ArrayList是可克隆的，因此该错误不会发生
            throw new InternalError();
        }
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 本方法是连接数组与集合的桥梁
     * 返回一个包含arrayList所有元素并有arrayList有相同顺序的数组
     * 
     * 返回的数组是"安全的"
     * 其与arrayList之间不存在引用关系(换句话说，本方法必须声明一个新的数组)
     * 因此调用者可以自由的修改返回的数组
     */
    public Object[] toArray() {
        // 注3：Arrays.copyOf(r, i)
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 查询操作。
     * 本方法是连接集合与数组之间的桥梁。
     * 
     * 返回包含列表所有元素的数组。该数组的顺序与列表的固有顺序相同。返回数组的类型即为a的类型。
     * 具体规则的伪代码为：
     * if (a.length < list.size())
     *     不修改a，而是以a的类型新建长度为list.size()的数组并填入列表中的值，随后返回这个新生成的数组。
     * else if (a.length == list.size()
     *     将返回的结果直接填入a后返回a(若a中已有值，则a中的原值会被覆盖)。
     * else
     *     数组索引在[0, list.size() -1]的元素会被列表对应位置的元素覆盖。
     *     数组索引 == list.size()的元素会被置为null
     *     数组中后续元素(如果有的话)不变。
     * 
     * 若列表不允许包含空元素，则本方法此时可用来计算列表的长度：返回的数组第一次出现null的索引即为列表的长度。
     * 
     * 返回的数组与列表之间不存在引用关系(即使列表的底层就是基于数组实现的)，数组中的元素是列表中元素的浅拷贝。
     * 
     * 小例子：
     * String[] y = x.toArray(new String[0]);
     * x是一个元素类型为String的列表，则上述语句会将x中的元素依序复制一份浅拷贝到数组y。
     * 
     * 关于本方法与前文介绍的toArray()方法，需明确：
     * list.toArray()
     * list.toArray(new Object[s])
     * 当s < list.size()时，上述两行代码的效果等价。
     * 
     * 本方法与前文介绍的toArray()方法很相似，可对比学习。
     * 
     * @throws ArrayStoreException a的类型列表不支持。
     * @throws NullPointerException null==a。
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // # 注1:Arrays.copyOf(r, i, newType)
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        // # 注4:System.arraycopy(r, 0, a, 0, i)
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }


    /**
     * 位置访问操作
     */
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 位置访问操作
     * 返回arrayList中索引位置为index的元素
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 以element替换arrayList中索引为index的元素。方法返回被替换的元素
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)
     */
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 将e添加至arrayList末尾
     */
    public boolean add(E e) {
        // 该方法内部会增加modCount
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 将element插入arrayList的index下标处
     * 原来处于index下标及以后的元素均向后移动一个位置
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        // 该方法内部会增加modCount
        ensureCapacityInternal(size + 1);
        // # 注4:System.arraycopy(r, 0, a, 0, i)
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 移除arrayList中索引值为index的元素
     * 后续元素左移一个位置
     * 返回被移除的元素
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)
     */
    public E remove(int index) {
        rangeCheck(index);

        // # 注2:modCount
        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            // # 注4:System.arraycopy(r, 0, a, 0, i)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null;    // 便于GC回收

        return oldValue;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 移除o在arrayList中第一次出现的位置的元素
     * 如果arrayList中不包含o则arrayList将不会被本方法改变
     * 
     * 更一般的来说，移除如下元素：
     * o==null ? get(i)==null : o.equals(get(i))
     * 其中为i满足相等条件的最小索引值
     * 
     * 若本方法改变了arrayList则返回true，反之返回false
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    private void fastRemove(int index) {
        // # 注2:modCount
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            // # 注4:System.arraycopy(r, 0, a, 0, i)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null;    // 便于GC回收
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 移除arrayList中的所有元素
     * 调用本方法后arrayList将变为空arrayList
     */
    public void clear() {
        // # 注2:modCount
        modCount++;

        // 之所以不直接将elementData置为空
        // 是为了保持容积不变，同时便于GC回收
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 将c中所有元素插入arrayList的末尾，插入顺序为c的迭代器取出的顺序
     * 
     * 本方法并未定义如下事件发生时的解决策略：
     * 在将c中的元素添加至arrayList的过程中c发生变化
     * 这也意味着如下事件的解决策略同样未定义：
     * 将一个非空arrayList添加至自身
     * 
     * 若arrayList因本方法发生变化则返回true，反之返回false
     * 
     * @throws NullPointerException c==null
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);    // 增加modCount
        // # 注4:System.arraycopy(r, 0, a, 0, i)
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 将c中所有元素插入arrayList的指定位置(index)
     * 插入顺序为c的迭代器取出的顺序
     * 原index位置及以后位置的元素顺次向后移动，空出装载c的位置
     * 
     * 若arrayList因本方法发生变化则返回true，反之返回false
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)
     * @throws NullPointerException c==null
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);    // 增加modCount

        int numMoved = size - index;
        if (numMoved > 0)
            // # 注4:System.arraycopy(r, 0, a, 0, i)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 从arrayList中移除索引为[fromIndex,toIndex)的元素
     * 并将后续元素左移
     * 本方法减少了arrayList (toIndex - fromIndex)个元素
     * 若toIndex==fromIndex，本方法不产生实际效果
     * 
     * @throws IndexOutOfBoundsException 索引越界(fromIndex < 0 || fromIndex >= size() || toIndex > size() || toIndex < fromIndex)
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        // # 注4:System.arraycopy(r, 0, a, 0, i)
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        int newSize = size - (toIndex-fromIndex);
        // 便于GC回收
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 检查index是否在边界之内
     * 若不在，则抛出一个适当的运行时异常
     * 
     * 本方法不会检查index<0的情况：
     * 因为访问数组时索引>=0是前提条件
     * 若index<0，则数组会自行抛出ArrayIndexOutOfBoundsException
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * add及addAll方法所使用的边界检查方法
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 构建一个IndexOutOfBoundsException的详细信息
     * 在众多可能的错误处理代码中
     * 这种"溢出"的处理代码在server及client JVM中均有最佳的性能
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 从arrayList中移除所有与c的交集元素(求差集)
     * 若arrayList因本方法发生变化则返回true，反之返回false
     * 
     * @throws ClassCastException arrayList中至少有一个元素的类型与c不相容
     * @throws NullPointerException arrayList中至少有一个元素为null且c不允许null存在；或c==null
     */
    public boolean removeAll(Collection<?> c) {
        // # 注5:batchRemove(c, b)
        return batchRemove(c, false);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 保留arrayList与c的交集元素
     * 若arrayList因本方法发生变化则返回true，反之返回false
     * 
     * @throws ClassCastException arrayList中至少有一个元素的类型与c不相容
     * @throws NullPointerException arrayList中至少有一个元素为null且c不允许null存在；或c==null
     */
    public boolean retainAll(Collection<?> c) {
        // # 注5:batchRemove(c, b)
        return batchRemove(c, true);
    }

    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)    // complement == true：移除arrayList中与c不同的元素，retainAll调用。
                                                                 // complement == false：移除arrayList中与c相同的元素，removeAll调用。
                    elementData[w++] = elementData[r];
        } finally {
            // 在c.contains()抛出异常时兼容AbstractCollection的保护性行为
            if (r != size) {    // 进入此if说明try中的for循环未完成，即c.contains()在循环过程中抛出了异常
                                // 此时从抛出异常的索引值r起的后续元素均不应再参与remove行为了，即索引在[r,size-1]之间的元素原样保留。
                // # 注4:System.arraycopy(r, 0, a, 0, i)
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {    // 进入此if说明确实成功移除了元素。
                                // 则应标明已移除并修改size等属性。
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }

    /**
     * 将arrayList的状态保存至流中(序列化时使用)
     * 
     * 流中存储的信息为：
     * 1. arrayList本身的信息。
     * 2. arrayList的长度。
     * 3. arrayList中的元素按序排列。
     * 
     * @throws java.io.IOException
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // # 注2:modCount
        int expectedModCount = modCount;
        s.defaultWriteObject();    // arrayList本身的信息

        s.writeInt(size);    // 长度

        for (int i=0; i<size; i++) {    // arrayList中的元素
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    /**
     * 反序列化时从流中恢复arrayList
     * 
     * @throws java.io.IOException
     * @throws ClassNotFoundException
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        s.defaultReadObject();    // arrayList本身的信息

        s.readInt();    // 长度

        if (size > 0) {
            // 类似于clone()，声明所需大小的数组
            ensureCapacityInternal(size);

            Object[] a = elementData;
            for (int i=0; i<size; i++) {    // 依次读入流中所有元素
                a[i] = s.readObject();
            }
        }
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回一个arrayList按固有顺序迭代的listIterator
     * 迭代开始时游标位于索引[index-1,index]之间
     * 
     * 即使用本方法得到listIterator后
     * 若第一次调用的是listIterator.next()，返回的元素的索引是index；
     * 同理，若第一次调用的是listIterator.previous()，返回的是索引为index-1的元素
     * 
     * 返回的listIterator遵循fail-fast原则
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回一个arrayList按固有顺序迭代的listIterator
     * 返回的listIterator遵循fail-fast原则
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回一个arrayList按固有顺序迭代的iterator
     * 返回的iterator遵循fail-fast原则
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * Iterator在ArrayList的最优实现
     */
    private class Itr implements Iterator<E> {

        /**
         * 游标位于元素[cursor-1,cursor]之间
         */
        int cursor;

        /**
         * 上一次操作的元素的索引
         * 即调用next()或previous()后赋以对应的值
         * 初始时或调用remove()或add(E e)后置为-1
         */
        int lastRet = -1;

        /**
         * iterator认为的arrayList发生的结构性变化的次数
         * 初始时同步自其所属的arrayList
         */
        int expectedModCount = modCount;

        /**
         * 实现接口:java.util.Iterator<E>
         */
        public boolean hasNext() {
            return cursor != size;
        }

        /**
         * 实现接口:java.util.Iterator<E>
         */
        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        /**
         * 实现接口:java.util.Iterator<E>
         */
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    /**
     * ListIterator在arrayList的最优实现
     */
    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         */
        public boolean hasPrevious() {
            return cursor != 0;
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         */
        public int nextIndex() {
            return cursor;
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         */
        public int previousIndex() {
            return cursor - 1;
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         */
        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         */
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         */
        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回arrayList的一部分，索引值区间：
     * [fromIndex,toIndex)
     * 特别的，若fromIndex==toIndex，则返回空arrayList(不是null)
     * 
     * 返回的arrayList是原arrayList的一个视图
     * 前者是后者的一部分，并未从后者中分离出去
     * 因此作用在前者之上的修改会反映在后者上，反之亦然
     * 
     * 使用本方法的好处在于
     * 当我们操作arrayList的一部分时
     * 可以直接在这一部分内部操作(仿佛这就是一个新的arrayList)
     * 而无需被各种索引的边界分散过多的注意
     * 
     * 例如，如果想批量移除arrayList中的一部分，可按如下操作:
     * arrayList.subList(from, to).clear();
     * 
     * 同理，所有本类的方法及所有
     * java.util.Collections类
     * 提供的静态方法均支持subList取得的视图作为一个独立的arrayList调用
     * 例如:
     * arrayList2 = arrayList.subList(1, 2);
     * arrayList2.get(0);
     * 此时取得的元素即为arrayList.get(1)
     * 
     * 若在生成视图后原arrayList发生了结构性的变化(例如长度发生了变化)
     * 则已生成的视图将被重置为未定义的状态
     * 
     * @throws IndexOutOfBoundsException 索引越界(fromIndex < 0 || toIndex > size)
     * @throws IllegalArgumentException fromIndex > toIndex
     */
    public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }

    static void subListRangeCheck(int fromIndex, int toIndex, int size) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > size)
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
    }

    private class SubList extends AbstractList<E> implements RandomAccess {
        /**
         * 本视图的父亲list
         * 注意:
         * parent代表的是近亲父亲，也就是说依然可能是个视图，而非始祖
         * 
         * 例如始祖list生成了视图sublist1
         * sublist1生成了视图sublist2
         * sublist1的parent为list
         * sublist2的parent为sublist1
         */
        private final AbstractList<E> parent;

        /**
         * 本视图相对于父亲list的位移量
         * 
         * 例如有始祖list
         * list使用subList(f, t)得到了subList1
         * 则subList1的parent为list，parentOffset=f
         */
        private final int parentOffset;

        /**
         * 本视图相对于始祖的位移量
         */
        private final int offset;

        /**
         * 本视图长度
         */
        int size;

        /**
         * @param parent AbstractList<E>, 父亲list
         * @param offset int, 父亲list相对于始祖的位移
         * @param fromIndex int, 本视图相对于父亲list的位移
         * @param toIndex int, 本视图相对于父亲list的位移结束量。实际本视图包含的元素为父亲list中的[fromIndex,toIndex)
         */
        SubList(AbstractList<E> parent,
                int offset, int fromIndex, int toIndex) {
            this.parent = parent;
            this.parentOffset = fromIndex;
            this.offset = offset + fromIndex;
            this.size = toIndex - fromIndex;
            this.modCount = ArrayList.this.modCount;
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         */
        public E set(int index, E e) {
            rangeCheck(index);
            checkForComodification();
            E oldValue = ArrayList.this.elementData(offset + index);
            ArrayList.this.elementData[offset + index] = e;
            return oldValue;
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         */
        public E get(int index) {
            rangeCheck(index);
            checkForComodification();
            return ArrayList.this.elementData(offset + index);
        }

        public int size() {
            checkForComodification();
            return this.size;
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         * 
         * 本方法之所以使用了parent的add方法而非始祖的add方法
         * 是因为本方法使始祖list发生了结构性变化
         * 这种变化必须通知到本视图的所有前辈
         */
        public void add(int index, E e) {
            rangeCheckForAdd(index);
            checkForComodification();
            parent.add(parentOffset + index, e);
            this.modCount = parent.modCount;
            this.size++;
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         * 
         * 本方法之所以使用了parent的remove方法而非始祖的remove方法
         * 是因为本方法使始祖list发生了结构性变化
         * 这种变化必须通知到本视图的所有前辈
         */
        public E remove(int index) {
            rangeCheck(index);
            checkForComodification();
            E result = parent.remove(parentOffset + index);
            this.modCount = parent.modCount;
            this.size--;
            return result;
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         * 
         * 本方法之所以使用了parent的removeRange方法而非始祖的removeRange方法
         * 是因为本方法使始祖list发生了结构性变化
         * 这种变化必须通知到本视图的所有前辈
         */
        protected void removeRange(int fromIndex, int toIndex) {
            checkForComodification();
            parent.removeRange(parentOffset + fromIndex,
                               parentOffset + toIndex);
            this.modCount = parent.modCount;
            this.size -= toIndex - fromIndex;
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         */
        public boolean addAll(Collection<? extends E> c) {
            return addAll(this.size, c);
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         * 
         * 本方法之所以使用了parent的addAll方法而非始祖的addAll方法
         * 是因为本方法使始祖list发生了结构性变化
         * 这种变化必须通知到本视图的所有前辈
         */
        public boolean addAll(int index, Collection<? extends E> c) {
            rangeCheckForAdd(index);
            int cSize = c.size();
            if (cSize==0)
                return false;

            checkForComodification();
            parent.addAll(parentOffset + index, c);
            this.modCount = parent.modCount;
            this.size += cSize;
            return true;
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         */
        public Iterator<E> iterator() {
            // 本类并未重写listIterator()方法
            // 因此这里调用的是父类AbstractList的listIterator()方法
            // 父类AbstractList的listIterator()方法实际调用的是listIterator(0)
            // 而本类重写了listIterator(i)
            // 故最终实际调用的是本类的listIterator(i)
            return listIterator();
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         * 
         * 入参index是相对本视图内部的索引值
         */
        public ListIterator<E> listIterator(final int index) {
            checkForComodification();
            rangeCheckForAdd(index);
            final int offset = this.offset;

            return new ListIterator<E>() {
                int cursor = index;
                int lastRet = -1;
                int expectedModCount = ArrayList.this.modCount;

                public boolean hasNext() {
                    return cursor != SubList.this.size;
                }

                @SuppressWarnings("unchecked")
                public E next() {
                    checkForComodification();
                    int i = cursor;
                    if (i >= SubList.this.size)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i + 1;
                    return (E) elementData[offset + (lastRet = i)];
                }

                public boolean hasPrevious() {
                    return cursor != 0;
                }

                @SuppressWarnings("unchecked")
                public E previous() {
                    checkForComodification();
                    int i = cursor - 1;
                    if (i < 0)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i;
                    return (E) elementData[offset + (lastRet = i)];
                }

                public int nextIndex() {
                    return cursor;
                }

                public int previousIndex() {
                    return cursor - 1;
                }

                public void remove() {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        SubList.this.remove(lastRet);
                        cursor = lastRet;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void set(E e) {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        ArrayList.this.set(offset + lastRet, e);
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void add(E e) {
                    checkForComodification();

                    try {
                        int i = cursor;
                        SubList.this.add(i, e);
                        cursor = i + 1;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                final void checkForComodification() {
                    if (expectedModCount != ArrayList.this.modCount)
                        throw new ConcurrentModificationException();
                }
            };
        }

        /**
         * 重写父类:java.util.AbstractList<E>
         */
        public List<E> subList(int fromIndex, int toIndex) {
            subListRangeCheck(fromIndex, toIndex, size);
            return new SubList(this, offset, fromIndex, toIndex);
        }

        private void rangeCheck(int index) {
            if (index < 0 || index >= this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private void rangeCheckForAdd(int index) {
            if (index < 0 || index > this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private String outOfBoundsMsg(int index) {
            return "Index: "+index+", Size: "+this.size;
        }

        private void checkForComodification() {
            if (ArrayList.this.modCount != this.modCount)
                throw new ConcurrentModificationException();
        }
    }
}
```

# 已整理层级关系

***本类直接继承的类***

- [java.util.AbstractList&lt;E&gt;](/2017/06/19/Java JDK7源码-javautilAbstractListE/)

***本类直接实现的接口***

- [java.util.List&lt;E&gt;](/2017/05/25/Java JDK7源码-javautilListE/)
- [java.util.RandomAccess](/2017/06/22/Java JDK7源码-javautilRandomAccess/)
- [java.lang.Cloneable](/2017/07/04/Java JDK7源码-javalangCloneable/)
- [java.io.Serializable](/2017/07/04/Java JDK7源码-javaioSerializable/)

# 综述

本类是Java集合框架中的一员，底层基于数组实现了List接口。

本类实现了List接口所有可选的操作，允许包含null在内的所有元素。除了实现List接口外，本类还提供调整本类内部用于存储arrayList的数组的长度的方法。

除了Vector不具备序列化能力及本类不具备线程安全的能力外，粗略的讲，可以认为本类与Vector等价。

size()，isEmpty()，get(int index)，set(int index, E element)，iterator()，listIterator()时间复杂度为O(1)。add(E e)的时间复杂度为常数恒定分摊时间(添加一个元素为O(1)，若需扩容则需大于O(1)的时间)，即：添加n个元素需要O(n)的时间。所有其他的操作花费线性阶的时间(粗略的讲)。总体来说，时间复杂度较之LinkedList要低。

---

**capacity**

每一个本类的实例都有一个容量，容量是指被用于存储arrayList中的元素的数组的长度。它的长度总是至少等于arrayList的长度。随着元素被插入arrayList，该数组的长度会自动增长。只要满足添加一个元素的时间复杂度为常数恒定分摊时间即可，对于扩容策略的细节并未有强制规定。

使用时，可以在向list中添加大量元素前使用ensureCapacity(int minCapacity)对arrayList进行扩容。这样可以减少重新分配增量空间的次数。

**synchronized**

注意，本实现是线程不安全的。如果复数个线程同时访问一个arrayList实例，并且至少一个线程对arrayList做出了结构性修改(结构性操作是指任何添加或删除一个或一个以上元素的操作，或者更确切的说，改变了内部数组的大小。仅仅只是设置一个元素的值不是结构性变化)，则必须要在调用外部保证线程安全。通常来说，这种线程安全性需要容器本身来确保(例如Vector)，不过，如果实在是需要在并发环境下使用线程不安全的list，例如本类，则可用

```
Collections.synchronizedList(List<T> list)
```

封装。即：

```
List list = Collections.synchronizedList(new ArrayList(...));
```

该操作最好在声明arrayList时即进行，以避免意外情况下会有线程不安全的访问请求arrayList。

**fail-fast**

本类iterator()及listIterator()返回的迭代器采用快速失败模式：在迭代器创建后，若arrayList在任何时候以任何方式发生了结构性变化，除非原因是因为迭代器本身触发(具体来说：ListIterator.remove()及ListIterator.add(E e))，否则迭代器会抛出ConcurrentModificationException。因此，在面对并发性修改时，迭代器会简单明了的失败，而非依据实际的改变情况，在未来采取不确定的行为。

事实上，fail-fast模式并不能完全根除线程不安全的并发修改。它只是尽力而为：若实在力有不逮则迭代器会抛出ConcurrentModificationException。因此在程序中完全依靠fail-fast避免线程安全问题是错误的：fail-fast模式应该只被用于检测bug。

# 注1:Arrays.copyOf(r, i, newType)

```
Arrays.copyOf(U[] r, int i, Class<? extends T[]> newType)
```

以r为基础返回一个长度为i的数组，返回数组的类型为newType。

- 若i&lt;r.length则r发生截断。实际返回的数组为r中索引为[0,i-1]的元素。
- 若i==r.length，则返回的数组与r相等。
- 若i&gt;r.length，返回的数组中索引为[0, r.length-1]的元素为r中对应位置的元素，索引为[r.length, i-1]的元素以null填充。

无论如何，返回的数组都是新生成的，与r不存在引用关系。返回的数组中的元素是r中元素的浅拷贝。

# 注2:modCount

modCount字段继承自超类AbstractList：

list发生结构性变化的次数。结构性变化是指改变list的长度，或是其他会使迭代器结果混乱的变化。

本字段将被iterator()返回的iterator及listIterator()返回的listIterator使用。若本字段发生了 iterator/listIterator 所没有预期到的变化，则在调用 iterator/listIterator 的next()，remove()，previous()，set(E e)，add(E e)出现fail-fast时抛出ConcurrentModificationException。

在迭代过程中，若检测到并发性变化，则直接判定为失败并抛出异常(fail-fast)，而不会去进一步检测所发生的并发性变化是否真的会对迭代造成影响(non-deterministic)。

子类可自行选择是否使用本字段。若子类决定继承本类的fail-fast判定，则只需要在会引发结构性变化的方法中增加本字段的值。本类中已实现的引发结构性变化的方法有add(int index, E element)及remove(int index)。调用一次add(int index, E element)或remove(int index)只需将本字段自增1，表示发生了一次结构性变化，否则 iterator/listIterator 会抛出错误的ConcurrentModificationException。

若子类不想遵循fail-fast，忽略本字段即可。

# 注3:Arrays.copyOf(r, i)

以r为基础返回一个长度为i的数组。

- 若i&lt;r.length则r发生截断。实际返回的数组为r中索引为[0,i-1]的元素。
- 若i==r.length，则返回的数组与r相等。
- 若i&gt;r.length，返回的数组中索引为[0, r.length-1]的元素为r中对应位置的元素，索引为[r.length, i-1]的元素以null填充。

无论如何，返回的数组都是新生成的，与r不存在引用关系。返回的数组中的元素是r中元素的浅拷贝。

# 注4:System.arraycopy(r, 0, a, 0, i)

参数含义依次为：

1. r: 待复制的源数组。
2. 0: 源数组中开始复制的索引。
3. a: 复制目标数组。
4. 0: 目标数组中接收元素的起始索引。
5. i: 复制的元素个数。

目标数组中其他位置的元素不受影响。

# 注5:batchRemove(c, b)

第二个参数的类型为boolean，其含义为：

- true：移除arrayList中与c不同的元素。
- false：移除arrayList中与c相同的元素。