---
title: Java JDK7源码-java.util.Vector&lt;E&gt;
date: 2017-09-05 15:36:36
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{

    /**
     * vector用于存储元素的数组缓冲区
     * vector的容量(capacity)为该数组的长度
     * 其大小至少会满足能容纳vector的所有元素
     * 
     * 该数组中未存储元素的空间以null填充
     */
    protected Object[] elementData;

    /**
     * vector中实际存储元素的个数
     * 因此数组elementData中索引在
     * [0,elementCount-1]
     * 之间的元素为vector实际存储的元素
     */
    protected int elementCount;

    /**
     * 当vector容量不足时自动扩展的容量值
     * 若capacityIncrement<=0，则vector每次自动扩展时容量翻倍
     */
    protected int capacityIncrement;

    private static final long serialVersionUID = -2767605614048989439L;

    /**
     * @throws IllegalArgumentException initialCapacity<0
     */
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }

    /**
     * @throws IllegalArgumentException initialCapacity<0
     */
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }

    public Vector() {
        this(10);
    }

    /**
     * 以c为基础构造vector，vector中元素的顺序为c迭代器返回的顺序
     * 
     * @throws NullPointerException c==null
     */
    public Vector(Collection<? extends E> c) {
        // c.toArray()可能不返回Object[](见6260652)
        // 说明：
        // 因为c可以是任意的Collection实现
        // 而Collection对toArray()的返回值仅仅是要求Object[]
        // 即实际的返回可以是任意Object的子类
        elementData = c.toArray();
        elementCount = elementData.length;
        if (elementData.getClass() != Object[].class)
            // 注1:Arrays.copyOf(r, i, newType)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }

    /**
     * 将vector中的元素copy至anArray中
     * vector中索引位置为k的元素会被copy至anArray的索引k处
     * 
     * @throws NullPointerException anArray==null
     * @throws IndexOutOfBoundsException anArray.length<vector.size()
     * @throws ArrayStoreException vector中至少有一个元素因类型不符无法被存入anArray
     */
    public synchronized void copyInto(Object[] anArray) {
        // 注2:System.arraycopy(r, 0, a, 0, i)
        System.arraycopy(elementData, 0, anArray, 0, elementCount);
    }

    /**
     * 裁剪vector的容量为list当前的实际长度
     * 应用可以使用本方法达到vector的存储空间占用最小化
     */
    public synchronized void trimToSize() {
        // 注3:modCount
        modCount++;
        int oldCapacity = elementData.length;
        if (elementCount < oldCapacity) {
            // 注4:Arrays.copyOf(r, i)
            elementData = Arrays.copyOf(elementData, elementCount);
        }
    }

    /**
     * 保证vector的容量足以容纳最少minCapacity个元素
     * 如果有必要(即vector的容量不足以容纳最少minCapacity个元素)
     * 则增大vector的容量
     * 
     * 若vector的当前容量不足以容纳minCapacity个元素
     * 则替换vector的内部数据数组elementData为一个更大的数组：
     * if (capacityIncrement<=0) 新数组的容量较之老数组翻倍
     * if (capacityIncrement>0) 新数组的容量为老数组的容量加上capacityIncrement
     * 若如此扩展后的新数组容量仍无法容纳minCapacity个元素，则新数组的长度将被设置为minCapacity
     */
    public synchronized void ensureCapacity(int minCapacity) {
        if (minCapacity > 0) {
            // 注3:modCount
            modCount++;
            ensureCapacityHelper(minCapacity);
        }
    }

    /**
     * 本方法不是线程安全的
     * 本类中线程安全的方法可以在内部调用本方法以确保vector的容量满足需求
     * 同时不会导致额外的并发开销
     */
    private void ensureCapacityHelper(int minCapacity) {
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 数组可申请的最大容量
     * 某些JVM会在数组中保存一些头信息
     * 试图申请一个大容量的数组可能会导致OutOfMemoryError：
     * 即需要的数组容量超出JVM的限制
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 注4:Arrays.copyOf(r, i)
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
     * 设置vector的有效长度
     * 若newSize大于当前有效长度，则确保vector能容纳newSize个元素(不一定需要扩容)
     * 若newSize小于当前有效长度，所有索引大于等于newSize的元素都被置为null
     * 
     * @throws ArrayIndexOutOfBoundsException newSize<0
     */
    public synchronized void setSize(int newSize) {
        // 注3:modCount
        modCount++;
        if (newSize > elementCount) {
            ensureCapacityHelper(newSize);
        } else {
            for (int i = newSize ; i < elementCount ; i++) {
                elementData[i] = null;
            }
        }
        elementCount = newSize;
    }

    /**
     * 返回vector的容量
     * 即为vector内部存储数据的数组的长度
     * 不是vector的有效长度
     */
    public synchronized int capacity() {
        return elementData.length;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回vector的有效长度
     */
    public synchronized int size() {
        return elementCount;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 实现接口:java.util.List<E>
     * 
     * 若vector的有效长度为0则返回true，反之返回false
     */
    public synchronized boolean isEmpty() {
        return elementCount == 0;
    }

    public Enumeration<E> elements() {
        return new Enumeration<E>() {
            int count = 0;

            public boolean hasMoreElements() {
                return count < elementCount;
            }

            public E nextElement() {
                synchronized (Vector.this) {
                    if (count < elementCount) {
                        return elementData(count++);
                    }
                }
                throw new NoSuchElementException("Vector Enumeration");
            }
        };
    }

    public boolean contains(Object o) {
        return indexOf(o, 0) >= 0;
    }

    public int indexOf(Object o) {
        return indexOf(o, 0);
    }

    public synchronized int indexOf(Object o, int index) {
        if (o == null) {
            for (int i = index ; i < elementCount ; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    public synchronized int lastIndexOf(Object o) {
        return lastIndexOf(o, elementCount-1);
    }

    public synchronized int lastIndexOf(Object o, int index) {
        if (index >= elementCount)
            throw new IndexOutOfBoundsException(index + " >= "+ elementCount);

        if (o == null) {
            for (int i = index; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    public synchronized E elementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }

        return elementData(index);
    }

    public synchronized E firstElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(0);
    }

    public synchronized E lastElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(elementCount - 1);
    }

    public synchronized void setElementAt(E obj, int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        elementData[index] = obj;
    }

    public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null;
    }

    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }

    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }

    public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj);
        if (i >= 0) {
            removeElementAt(i);
            return true;
        }
        return false;
    }

    public synchronized void removeAllElements() {
        modCount++;
        for (int i = 0; i < elementCount; i++)
            elementData[i] = null;

        elementCount = 0;
    }

    public synchronized Object clone() {
        try {
            @SuppressWarnings("unchecked")
                Vector<E> v = (Vector<E>) super.clone();
            v.elementData = Arrays.copyOf(elementData, elementCount);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
    }

    public synchronized Object[] toArray() {
        return Arrays.copyOf(elementData, elementCount);
    }

    @SuppressWarnings("unchecked")
    public synchronized <T> T[] toArray(T[] a) {
        if (a.length < elementCount)
            return (T[]) Arrays.copyOf(elementData, elementCount, a.getClass());

        System.arraycopy(elementData, 0, a, 0, elementCount);

        if (a.length > elementCount)
            a[elementCount] = null;

        return a;
    }

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }

    public synchronized E set(int index, E element) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }

    public boolean remove(Object o) {
        return removeElement(o);
    }

    public void add(int index, E element) {
        insertElementAt(element, index);
    }

    public synchronized E remove(int index) {
        modCount++;
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);
        E oldValue = elementData(index);

        int numMoved = elementCount - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--elementCount] = null;

        return oldValue;
    }

    public void clear() {
        removeAllElements();
    }

    public synchronized boolean containsAll(Collection<?> c) {
        return super.containsAll(c);
    }

    public synchronized boolean addAll(Collection<? extends E> c) {
        modCount++;
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);
        System.arraycopy(a, 0, elementData, elementCount, numNew);
        elementCount += numNew;
        return numNew != 0;
    }

    public synchronized boolean removeAll(Collection<?> c) {
        return super.removeAll(c);
    }

    public synchronized boolean retainAll(Collection<?> c) {
        return super.retainAll(c);
    }

    public synchronized boolean addAll(int index, Collection<? extends E> c) {
        modCount++;
        if (index < 0 || index > elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);

        int numMoved = elementCount - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        elementCount += numNew;
        return numNew != 0;
    }

    public synchronized boolean equals(Object o) {
        return super.equals(o);
    }

    public synchronized int hashCode() {
        return super.hashCode();
    }

    public synchronized String toString() {
        return super.toString();
    }

    public synchronized List<E> subList(int fromIndex, int toIndex) {
        return Collections.synchronizedList(super.subList(fromIndex, toIndex),
                                            this);
    }

    protected synchronized void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = elementCount - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        int newElementCount = elementCount - (toIndex-fromIndex);
        while (elementCount != newElementCount)
            elementData[--elementCount] = null;
    }

    private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException {
        final java.io.ObjectOutputStream.PutField fields = s.putFields();
        final Object[] data;
        synchronized (this) {
            fields.put("capacityIncrement", capacityIncrement);
            fields.put("elementCount", elementCount);
            data = elementData.clone();
        }
        fields.put("elementData", data);
        s.writeFields();
    }

    public synchronized ListIterator<E> listIterator(int index) {
        if (index < 0 || index > elementCount)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    public synchronized ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    public synchronized Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        int cursor;
        int lastRet = -1;
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != elementCount;
        }

        public E next() {
            synchronized (Vector.this) {
                checkForComodification();
                int i = cursor;
                if (i >= elementCount)
                    throw new NoSuchElementException();
                cursor = i + 1;
                return elementData(lastRet = i);
            }
        }

        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            synchronized (Vector.this) {
                checkForComodification();
                Vector.this.remove(lastRet);
                expectedModCount = modCount;
            }
            cursor = lastRet;
            lastRet = -1;
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    final class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        public E previous() {
            synchronized (Vector.this) {
                checkForComodification();
                int i = cursor - 1;
                if (i < 0)
                    throw new NoSuchElementException();
                cursor = i;
                return elementData(lastRet = i);
            }
        }

        public void set(E e) {
            if (lastRet == -1)
                throw new IllegalStateException();
            synchronized (Vector.this) {
                checkForComodification();
                Vector.this.set(lastRet, e);
            }
        }

        public void add(E e) {
            int i = cursor;
            synchronized (Vector.this) {
                checkForComodification();
                Vector.this.add(i, e);
                expectedModCount = modCount;
            }
            cursor = i + 1;
            lastRet = -1;
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

Vector实现了一个可增长的数组对象。像数组一样，Vector可使用索引进行随机访问。但是Vector在创建后，其长度可随着添加或移除元素而增长或收缩。

vector试图通过维护capacity及capacityIncrement以达到存储管理的最优化。capacity总是至少和vector的长度一样大；capacity通常都会比vector的长度大一些，因为随着元素的添加vector是以capacityIncrement为基数成块增长。为减少空间重分配的次数，应用可以在添加大量元素前增大vector的capacity至一个合理的值。

vector返回的iterator：

```
iterator() : iterator
listIterator(int) : listIterator
```

遵循fail-fast原则：在iterator创建后，若vector因非该iterator的原因，即不是该iterator的以下方法：

```
ListIterator#remove() remove
ListIterator#add(Object) add
```

而发生了结构性变化，该iterator会抛出ConcurrentModificationException。因此，面对并发性修改，iterator放弃的快速而彻底，并不会去仔细考察该并发性修改是否真的会对自身即将进行的操作造成不利影响。vector的elements()返回的Enumeration对象不遵循fail-fast原则。

注意iterator遵循的fail-fast原则并不能从根本上解决并发问题，它仅仅只是尽力而为，若要避免并发问题，还需结构本身提供线程安全保护。因此因fail-fast原则抛出的ConcurrentModificationException仅应被用于检测bug，而非以其为依据进行并发保护。

在JDK1.2版本中，Vector被重写，重写后的Vector实现了List接口，因此其成为了Java集合框架中的一员。和新的集合实现类不同，Vector是线程安全的。若应用不需要线程安全的实现，推荐使用ArrayList替代本类。

# 注1:Arrays.copyOf(r, i, newType)

```
Arrays.copyOf(U[] r, int i, Class<? extends T[]> newType)
```

以r为基础返回一个长度为i的数组，返回数组的类型为newType。

- 若i&lt;r.length则r发生截断。实际返回的数组为r中索引为[0,i-1]的元素。

- 若i==r.length，则返回的数组与r相等。

- 若i&gt;r.length，返回的数组中索引为[0, r.length-1]的元素为r中对应位置的元素，索引为[r.length, i-1]的元素以null填充。

无论如何，返回的数组都是新生成的，与r不存在引用关系。返回的数组中的元素是r中元素的浅拷贝。

# 注2:System.arraycopy(r, 0, a, 0, i)

参数含义依次为：

1. r: 待复制的源数组。
2. 0: 源数组中开始复制的索引。
3. a: 复制目标数组。
4. 0: 目标数组中接收元素的起始索引。
5. i: 复制的元素个数。

目标数组中其他位置的元素不受影响。

# 注3:modCount

modCount字段继承自超类AbstractList：

list发生结构性变化的次数。结构性变化是指改变list的长度，或是其他会使迭代器结果混乱的变化。

本字段将被iterator()返回的iterator及listIterator()返回的listIterator使用。若本字段发生了 iterator/listIterator 所没有预期到的变化，则在调用 iterator/listIterator 的next()，remove()，previous()，set(E e)，add(E e)出现fail-fast时抛出ConcurrentModificationException。

在迭代过程中，若检测到并发性变化，则直接判定为失败并抛出异常(fail-fast)，而不会去进一步检测所发生的并发性变化是否真的会对迭代造成影响(non-deterministic)。

子类可自行选择是否使用本字段。若子类决定继承本类的fail-fast判定，则只需要在会引发结构性变化的方法中增加本字段的值。本类中已实现的引发结构性变化的方法有add(int index, E element)及remove(int index)。调用一次add(int index, E element)或remove(int index)只需将本字段自增1，表示发生了一次结构性变化，否则 iterator/listIterator 会抛出错误的ConcurrentModificationException。

若子类不想遵循fail-fast，忽略本字段即可。

# 注4:Arrays.copyOf(r, i)

以r为基础返回一个长度为i的数组。

- 若i&lt;r.length则r发生截断。实际返回的数组为r中索引为[0,i-1]的元素。

- 若i==r.length，则返回的数组与r相等。

- 若i&gt;r.length，返回的数组中索引为[0, r.length-1]的元素为r中对应位置的元素，索引为[r.length, i-1]的元素以null填充。

无论如何，返回的数组都是新生成的，与r不存在引用关系。返回的数组中的元素是r中元素的浅拷贝。