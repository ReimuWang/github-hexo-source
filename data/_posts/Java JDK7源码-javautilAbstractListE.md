---
title: Java JDK7源码-java.util.AbstractList&lt;E&gt;
date: 2017-06-19 17:26:41
tags: [Java,源码,Collection]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;

public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {

    protected AbstractList() {
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 实现接口:java.util.List<E>
     * 
     * 本方法可选
     * 将特定元素添加至list结尾。
     * 
     * 支持本方法的list可能会对可添加至list的元素有所限制。
     * 具体来说，某些实现拒绝添加空元素；某些实现会限制可添加元素的类型。
     * 程序员在继承本类时需在实现类中明确指明这些限制。
     * 
     * 本方法通过调用add(int index, E element)以实现自身功能。
     * 因add(int index, E element)默认抛出UnsupportedOperationException
     * 因此除非实现类重写了add(int index, E element)
     * 否则调用本方法时会抛出UnsupportedOperationException。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws ClassCastException e因为其所属的类禁止被插入list。
     * @throws NullPointerException e==null且list禁止包含空元素。
     * @throws IllegalArgumentException e因其某些属性禁止被插入list。
     */
    public boolean add(E e) {
        add(size(), e);
        return true;
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 返回list中索引位置为index的元素。
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    abstract public E get(int index);

    /**
     * 实现接口:java.util.List<E>
     * 
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 用element替换list索引值为index的元素。方法返回被替换的元素。
     * 本方法总是抛出UnsupportedOperationException。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws ClassCastException element因为其所属的类禁止被插入list。
     * @throws NullPointerException element==null且list禁止包含空元素。
     * @throws IllegalArgumentException element因其某些属性禁止被插入list。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    public E set(int index, E element) {
        throw new UnsupportedOperationException();
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 将element插入list的index下标处。原来处于index下标及以后的元素均向后移动一个位置。
     * 
     * list.add("1");
     * list.add(list.size(), "1");
     * 二者等效。
     * 
     * 本方法总是抛出UnsupportedOperationException。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws ClassCastException element因为其所属的类禁止被插入list。
     * @throws NullPointerException element==null且list禁止包含空元素。
     * @throws IllegalArgumentException element因其某些属性禁止被插入list。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 移除list中索引值为index的元素。后续元素左移一个位置。返回被移除的元素。
     * 
     * 本方法总是抛出UnsupportedOperationException。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    public E remove(int index) {
        throw new UnsupportedOperationException();
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 搜索操作
     * 返回o在list中第一次出现的索引。
     * 若list不包含o则返回-1。
     * 
     * 更一般的来说，返回满足如下条件索引值最小的元素的索引：
     * (o==null ? get(i)==null : o.equals(get(i)))。
     * 若不存在这样的元素则返回-1。
     * 
     * 本方法首先使用listIterator()获得一个list iterator。
     * 随后，向后迭代list直到找到o或迭代结束。
     * 
     * @throws ClassCastException o所属的类与list不相容(依实现可选)。
     * @throws NullPointerException o==null且list禁止包含空元素。
     */
    public int indexOf(Object o) {
        ListIterator<E> it = listIterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return it.previousIndex();
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return it.previousIndex();
        }
        return -1;
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 搜索操作
     * 返回o在list中最后一次出现的索引。
     * 若list不包含o则返回-1。
     * 
     * 更一般的来说，返回满足如下条件索引值最大的元素的索引：
     * (o==null ? get(i)==null : o.equals(get(i)))。
     * 若不存在这样的元素则返回-1。
     * 
     * 本方法首先使用listIterator(final int index)获得一个指向
     * [list.size()-1,list.size()]
     * 的list iterator。
     * 随后，向前迭代list直到找到o或迭代结束。
     * 
     * @throws ClassCastException o所属的类与list不相容(依实现可选)。
     * @throws NullPointerException o==null且list禁止包含空元素。
     */
    public int lastIndexOf(Object o) {
        ListIterator<E> it = listIterator(size());
        if (o==null) {
            while (it.hasPrevious())
                if (it.previous()==null)
                    return it.nextIndex();
        } else {
            while (it.hasPrevious())
                if (o.equals(it.previous()))
                    return it.nextIndex();
        }
        return -1;
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 实现接口:java.util.List<E>
     * 
     * 批量操作
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 
     * 移除list中的所有元素,调用本方法后list将为空。
     * 本方法通过调用removeRange(int fromIndex, int toIndex)以实现自身功能。
     * 因此除非重写removeRange(int fromIndex, int toIndex)或remove(int index)
     * 否则本方法将抛出UnsupportedOperationException，调用链条如下：
     * 
     * clear() ->
     * removeRange(int fromIndex, int toIndex) ->
     * listIterator.remove() ->
     * remove(int index)
     * 
     * remove(int index)默认抛出UnsupportedOperationException。
     */
    public void clear() {
        // removeRange(int fromIndex, int toIndex)说明:
        // 移除索引在[fromIndex,toIndex)范围内的元素
        removeRange(0, size());
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 本方法可选，实现类可依自身情况决定是否真的需要实现。
     * 
     * 将c中所有元素插入list的指定位置(index)，插入顺序为c的迭代器取出的顺序。
     * 原index位置及以后位置的元素顺次向后移动，空出装载c的位置。
     * 
     * 本方法并未定义如下事件发生时的解决策略：
     * 在将c中的元素添加至list的过程中c发生变化
     * 这也意味着如下事件的解决策略同样未定义：将一个非空list添加至自身。
     * 
     * 若list因本方法发生变化则返回true，反之返回false。
     * 
     * list.addAll(set);
     * list.addAll(list.size(), set);
     * 二者等价。
     * 
     * 本方法首先得到c的迭代器
     * 而后遍历c，将遍历得到的c中的元素插入list的索引index处
     * 具体方法为使用add(int index, E element)
     * 因为add(int index, E element)默认抛出UnsupportedOperationException
     * 因此如果要实现本方法的功能，则必须重写add(int index, E element)。
     * 
     * 基于效率考虑，许多实现会重写本方法。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws ClassCastException c中任意一个元素因为其所属的类禁止被插入list。
     * @throws NullPointerException c中任意一个元素为null且list禁止包含空元素；或c==null。
     * @throws IllegalArgumentException c中任意一个元素因其某些属性禁止被插入list。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        // 注1:rangeCheckForAdd(int index)
        rangeCheckForAdd(index);
        boolean modified = false;
        for (E e : c) {
            add(index++, e);
            modified = true;
        }
        return modified;
    }

    /**
     * 重写父类:java.util.AbstractCollection<E>
     * 实现接口:java.util.List<E>
     * 
     * 返回一个基于list的有序迭代器。
     * 本方法返回的迭代器是Iterator接口的一个简单实现
     * 基于本类的size()，get(int index)，remove(int index)。
     * 
     * 返回的iterator的iterator.remove()需调用remove(int index)
     * 而remove(int index)默认抛出UnsupportedOperationException
     * 因此除非重写remove(int index)，否则在调用iterator.remove()时会抛出UnsupportedOperationException。
     * 
     * 基于protected字段modCount(list发生结构性变化的次数)的值
     * 本方法返回的iterator可能会抛出运行时异常(发生结构性变化的次数超过iterator的预期，即发生结构性变化不是由该iterator导致的)。
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 返回一个list按固有顺序迭代的listIterator。
     */
    public ListIterator<E> listIterator() {
        return listIterator(0);
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 返回一个list按固有顺序迭代的listIterator
     * 迭代开始时游标位于索引[index-1,index]之间。
     * 
     * 使用本方法得到listIterator后
     * 若第一次调用的是listIterator.next()，返回的元素的索引是index
     * 同理，若第一次调用的是listIterator.previous()，返回的是索引为index-1的元素。
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    public ListIterator<E> listIterator(final int index) {
        // 注1:rangeCheckForAdd(int index)
        rangeCheckForAdd(index);

        return new ListItr(index);
    }

    private class Itr implements Iterator<E> {

        /**
         * 当前游标指向的位置的后一个元素的索引。
         * 默认游标位于第一个元素之前。因此默认值为0
         */
        int cursor = 0;

        /**
         * 记录最近一次调用next()或previous()返回的元素的索引。
         * 若该元素被remove()移除则本值被重置为-1。
         * 初始时本值为-1。
         */
        int lastRet = -1;

        /**
         * 本字段记录了iterator认为的abstractList应有的结构性变化次数。
         * 初始时，本字段等于创建iterator那个时间点的modCount(modCount中记录的是list总的结构性变化次数)。
         * 此后凡是经由iterator导致的list的结构性变化
         * iterator均会同步修改expectedModCount及modCount，以保证二者始终相等。
         * 因此一旦iterator检测到了expectedModCount != modCount
         * 则证明list因为本iterator以外的原因发生了结构性变化
         * 进而可认为list存在同步性问题。
         */
        int expectedModCount = modCount;

        /**
         * 实现接口:java.util.Iterator<E>
         * 
         * 在迭代过程中，若集合还有元素则返回true。
         * 换句话说，返回ture说明仍然可以通过next()方法取到元素
         * 若在本方法返回false时继续调用next()方法，则next()方法会抛出异常。
         */
        public boolean hasNext() {
            return cursor != size();
        }

        /**
         * 实现接口:java.util.Iterator<E>
         * 返回iterator游标所指向的元素，同时游标后移一位
         * 注2:checkForComodification()
         * 
         * @throws NoSuchElementException iterator游标所指的位置已无元素。
         */
        public E next() {
            checkForComodification();
            try {
                int i = cursor;
                E next = get(i);
                lastRet = i;
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        /**
         * 实现接口:java.util.Iterator<E>
         * 
         * 移除当前游标所指元素的前一个元素。
         * 即上一次调用next()所返回的元素。
         * 因此，调用本方法之前必须调用一次next()。
         * 调用一次next()后多次调用本方法是不允许的。本方法必须与next()一一配对。
         * 
         * 本方法没有定义应对以下事件的对策：
         * 在迭代过程中，被迭代集合因本方法以外的原因发生了改变。
         * 
         * 注2:checkForComodification()
         * 注3:Itr.remove()测试
         * 
         * @throws UnsupportedOperationException iterator不支持本方法。
         * @throws IllegalStateException 在调用本方法前没有调用与之配对的next()(包含调用一次next()后多次调用本方法的情况)。
         */
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                // 本类Itr中只有hasNext()及next()，因此只会向后迭代
                // 如果仅考虑本类的话，是不需要这个if判断的
                // 因为必有lastRet < cursor，直接全部调用cursor--;
                // 然后若考虑到后文出现的，本类的子类ListItr的话，其内部有向前移动的hasPrevious()及previous()
                // 在向前移动的情况下，cursor < lastRet，此时remove()后无需做任何操作
                // 实在是精妙的操作，这样ListItr就无需重写本方法了
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        /**
         * 注2:checkForComodification()
         */
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            cursor = index;
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         * 
         * 当向前遍历还有元素时返回true。
         * 换句话说，返回ture说明仍然可以通过previous()取到元素
         * 若在本方法返回false时继续调用previous()，则previous()会抛出异常。
         */
        public boolean hasPrevious() {
            return cursor != 0;
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         * 
         * 返回listIterator当前游标前一个元素，同时游标前移一位。
         * 在迭代list的过程中本方法可能会被反复调用，或者与next()交替被调用。以此控制向前或向后迭代。
         * 
         * 以下语句均会返回当前游标的后一个元素：
         * listIterator.next();
         * listIterator.previous();
         * 
         * 以下语句均会返回当前游标的前一个元素：
         * listIterator.previous();
         * listIterator.next();
         * 
         * 注2:checkForComodification()
         * 
         * @throws NoSuchElementException 游标所指的位置之前已无元素。
         */
        public E previous() {
            checkForComodification();
            try {
                int i = cursor - 1;
                E previous = get(i);
                lastRet = cursor = i;
                return previous;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         * 
         * 返回listIterator当前游标后一个元素的索引。
         * 本操作不会移动游标。
         * 即本方法会返回下次调用next()时返回的元素的索引。
         * 特别的，当游标位于list末尾时，调用本方法不会抛出异常，而是会返回list的长度。
         */
        public int nextIndex() {
            return cursor;
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         * 
         * 返回listIterator当前游标前一个元素的索引。
         * 本操作不会移动游标。
         * 即本方法会返回下次调用previous()时返回的元素的索引。
         * 特别的，当游标位于list最左边时，调用本方法不会抛出异常，而是会返回-1。
         */
        public int previousIndex() {
            return cursor-1;
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         * 
         * 用e覆盖上一次由next()或previous()返回的元素的位置的元素。
         * 本方法在调用前必须调用一次next()或previous()，否则不知道该覆盖哪个位置的元素。
         * 且在本方法与最后一次next()或previous()之间不能调用add(E e)或remove()。
         * 
         * 本方法不会修改lastRet，因此自然也不会影响add(E e)或remove()
         * 但由于add(E e)或remove()均会修改lastRet为-1，因此这两个方法会影响本方法
         * 
         * 注2:checkForComodification()
         * 
         * @throws UnsupportedOperationException listIterator不支持本方法。
         * @throws ClassCastException e因为其所属的类禁止被插入list。
         * @throws IllegalArgumentException e因其某些属性禁止被插入list。
         * @throws IllegalStateException 本方法在调用前没有调用next()或previous()，或在本方法与最后一次next()或previous()之间调用add(E e)或remove()。
         */
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.set(lastRet, e);
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        /**
         * 实现接口:java.util.ListIterator<E>
         * 
         * 将e插入当前游标所在位置。若list为空，则e将成为list的第一个元素。
         * 
         * e插入后，当前游标位于e之后的位置。
         * 因此插入后第一次如果调用的是next()将返回插入前游标所指的下一个元素
         * 第一次如果调用的是previous()将返回e。
         * 
         * 调用该方法后，若调用nextIndex()或previousIndex()，和未调用该方法之前相比值均会增加1。
         * 
         * 注2:checkForComodification()
         * 
         * @throws UnsupportedOperationException listIterator不支持本方法。
         * @throws ClassCastException e因为其所属的类禁止被插入list。
         * @throws IllegalArgumentException e因其某些属性禁止被插入list。
         */
        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                AbstractList.this.add(i, e);
                lastRet = -1;
                cursor = i + 1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }

    /**
     * 实现接口:java.util.List<E>
     * 
     * 视图
     * 
     * 返回list的一部分，索引值区间：[fromIndex,toIndex)。
     * 特别的，若fromIndex==toIndex，则返回空list(不是null)。
     * 
     * 返回的列表是原list的一个视图，前者是后者的一部分，并未从后者中分离出去。
     * 因此作用在前者之上的修改会反映在后者上，反之亦然。
     * 
     * 使用本方法的好处在于：
     * 当我们操作list的一部分时，可以直接在这一部分内部操作(仿佛这就是一个新的list)，
     * 而无需被各种索引的边界分散过多的注意。
     * 
     * 例如，如果想批量移除list中的一部分，可按如下操作：
     * list.subList(fromIndex, toIndex).clear();
     * 
     * 同理，所有本接口的方法及所有java.util.Collections类提供的静态方法
     * 均支持本方法返回的视图作为一个独立的list调用。例如：
     * list2 = list.subList(1, 2);
     * list2.get(0);
     * 此时取得的元素即为list.get(1)。
     * 
     * 若在生成视图后原list发生了结构性的变化(例如长度发生了变化)
     * 则已生成的视图将被重置为未定义的状态。
     * 
     * 返回的视图在其私有域中存储了：
     * 1.相对于原list的偏移量
     * 2.视图的长度(该长度在视图生命周期中可变)
     * 3.原list的modCount(即原list发生结构性变化的次数)
     * 
     * 视图有两个变种：
     * 若原list实现了java.util.RandomAccess接口，则返回的视图将基于RandomAccessSubList构造
     * 反之，返回的视图将基于SubList构造。
     * 
     * 在SubList及RandomAccessSubList中：
     * 
     * 在越界检查及偏移量调整后，视图的：
     * set(int index, E element)
     * get(int index)
     * add(int index, E element)
     * remove(int index)
     * addAll(int index, Collection<? extends E> c)
     * removeRange(int fromIndex, int toIndex)
     * 均基于原list计算。
     * addAll(Collection<? extends E> c)是通过addAll(size, c)实现的。
     * 
     * listIterator(final int index)返回一个封装的对象
     * 该对象基于原list的listIterator，并附加视图的偏移量。
     * 
     * iterator()是通过调用listIterator()实现的。
     * 
     * size()是通过调用视图的size字段实现的。
     * 
     * 所有方法都会检查原list当前的modCount值(原list发生结构性变化的次数)是否与视图的预期相等。
     * 若不相等则抛出ConcurrentModificationException。
     * 
     * @throws IndexOutOfBoundsException 索引越界(fromIndex < 0 || toIndex > size)。
     * @throws IllegalArgumentException fromIndex > toIndex。
     */
    public List<E> subList(int fromIndex, int toIndex) {
        return (this instanceof RandomAccess ?
                new RandomAccessSubList<>(this, fromIndex, toIndex) :
                new SubList<>(this, fromIndex, toIndex));
    }

    /**
     * 重写祖先类:java.lang.Object
     * 实现接口:java.util.List<E>
     * 
     * 比较与哈希
     * 比较o与list是否相等。
     * 
     * 当且仅当:
     * o同样是一个列表
     * o与list长度相同
     * o与list所有对应位置的元素均相等
     * 时，认为o与list相等
     * 
     * 关于"o与list所有对应位置的元素均相等"，具体来说：
     * 若设e1为o中的某个元素，e2为list中对应相同位置的元素
     * 若有
     * e1==null ? e2==null : e1.equals(e2)
     * 则有e1与e2相等。
     * 换句话说，两个list相等的条件为：在相同的位置包含相同的元素。
     * 
     * 本方法首先检查o是否是list本身，如果是则返回true。
     * 若不是则检查o是否是一个list，若不是则返回false。
     * 如果是则同时迭代o及list，比较对应位置的元素。
     * 若任意某个比较结果返回false，返回false。
     * 若其中一个迭代器先于另一个迭代结束(即o与list长度不同)，返回false。
     * 若迭代正常结束，则返回true。
     */
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof List))
            return false;

        ListIterator<E> e1 = listIterator();
        ListIterator e2 = ((List) o).listIterator();
        while (e1.hasNext() && e2.hasNext()) {
            E o1 = e1.next();
            Object o2 = e2.next();
            if (!(o1==null ? o2==null : o1.equals(o2)))
                return false;
        }
        return !(e1.hasNext() || e2.hasNext());
    }

    /**
     * 重写祖先类:java.lang.Object
     * 实现接口:java.util.List<E>
     * 
     * 比较与哈希
     * 
     * 返回list的hash code值。
     * 计算方式如下：
     * int hashCode = 1;
     * for (E e : abstractList)
     *     hashCode = 31 * hashCode + (e == null ? 0 : e.hashCode());
     * 
     * 这种计算方式可以保证:
     * 任意list1.equals(list2)
     * 有
     * list1.hashCode()==list2.hashCode()。
     */
    public int hashCode() {
        int hashCode = 1;
        for (E e : this)
            hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
        return hashCode;
    }

    /**
     * 移除list索引区间为[fromIndex,toIndex)的全部元素。
     * 并将后续元素左移补位。
     * 本方法减少了list中(toIndex - fromIndex)个元素。
     * 若有toIndex==fromIndex，则本方法实际并未对list产生影响。
     * 
     * 本方法会被list及其视图的clear()调用。
     * 重写本方法可大幅提高clear()的性能。
     * 
     * 本方法首先得到一个listIterator
     * 并将初始游标定位于[fromIndex-1,fromIndex]。
     * 随后反复调用listIterator.next()及listIterator.remove()
     * 直到指定范围内的元素全部被移除。
     * 
     * 注意：
     * 若listIterator.remove()需要线性阶时间
     * 则本方法需要平方阶时间。
     */
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }

    /**
     * list发生结构性变化的次数。
     * 结构性变化是指改变list的长度，或是其他会使迭代器结果混乱的变化。
     * 
     * 本字段将被iterator()返回的iterator及listIterator()返回的listIterator使用。
     * 若本字段发生了 iterator/listIterator 所没有预期到的变化
     * 则在调用 iterator/listIterator 的:
     * next()
     * remove()
     * previous()
     * set(E e)
     * add(E e)
     * 出现fail-fast时抛出ConcurrentModificationException。
     * 
     * 在迭代过程中，若检测到并发性变化
     * 则直接判定为失败并抛出异常(fail-fast)
     * 而不会去进一步检测所发生的并发性变化是否真的会对迭代造成影响(non-deterministic)。
     * 
     * 子类可自行选择是否使用本字段。
     * 若子类决定继承本类的fail-fast判定
     * 则只需要在会引发结构性变化的方法中增加本字段的值。
     * 本类中已实现的引发结构性变化的方法有:
     * add(int index, E element)
     * remove(int index)
     * 
     * 调用一次add(int index, E element)或remove(int index)
     * 只需将本字段自增1，表示发生了一次结构性变化
     * 否则 iterator/listIterator 会抛出错误的ConcurrentModificationException。
     * 
     * 若子类不想遵循fail-fast，忽略本字段即可。
     */
    protected transient int modCount = 0;

    /**
     * 注1:rangeCheckForAdd(int index)
     */
    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size())
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size();
    }
}

class SubList<E> extends AbstractList<E> {
    /**
     * 原list
     */
    private final AbstractList<E> l;
    /**
     * 生成视图时的偏移量
     * 即原list.subList(int fromIndex, int toIndex)中的fromIndex。
     */
    private final int offset;
    /**
     * 视图包含元素数
     * 初始时为原list.subList(int fromIndex, int toIndex)中的toIndex-fromIndex。
     */
    private int size;

    SubList(AbstractList<E> list, int fromIndex, int toIndex) {
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > list.size())
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                               ") > toIndex(" + toIndex + ")");
        l = list;
        offset = fromIndex;
        size = toIndex - fromIndex;
        this.modCount = l.modCount;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 用element替换视图索引值(视图内部的相对值)为index的元素。
     * 方法返回被替换的元素。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws ClassCastException element因为其所属的类禁止被插入list。
     * @throws NullPointerException element==null且list禁止包含空元素。
     * @throws IllegalArgumentException element因其某些属性禁止被插入list。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    public E set(int index, E element) {
        // 注4:rangeCheck(index)
        rangeCheck(index);
        // 注2:checkForComodification()
        checkForComodification();
        return l.set(index+offset, element);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 返回视图中索引(视图内部的相对值)位置为index的元素。
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    public E get(int index) {
        // 注4:rangeCheck(index)
        rangeCheck(index);
        // 注2:checkForComodification()
        checkForComodification();
        return l.get(index+offset);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     */
    public int size() {
        // 注2:checkForComodification()
        checkForComodification();
        return size;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 将element插入视图的index(视图内部的相对值)下标处。原来处于index下标及以后的元素均向后移动一个位置。
     * 
     * 视图.add("1");
     * 视图.add(视图.size(), "1");
     * 二者等效。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws ClassCastException element因为其所属的类禁止被插入list。
     * @throws NullPointerException element==null且list禁止包含空元素。
     * @throws IllegalArgumentException element因其某些属性禁止被插入list。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    public void add(int index, E element) {
        // 注1:rangeCheckForAdd(int index)
        rangeCheckForAdd(index);
        // 注2:checkForComodification()
        checkForComodification();
        l.add(index+offset, element);
        this.modCount = l.modCount;
        size++;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 移除视图中索引值为index(视图内部的相对值)的元素。后续元素左移一个位置。返回被移除的元素。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index >= size)。
     */
    public E remove(int index) {
        // 注4:rangeCheck(index)
        rangeCheck(index);
        // 注2:checkForComodification()
        checkForComodification();
        E result = l.remove(index+offset);
        this.modCount = l.modCount;
        size--;
        return result;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 后文注释讨论的list均是指视图，其中的index均是视图中的相对值
     * 
     * 移除list索引区间为[fromIndex,toIndex)的全部元素。
     * 并将后续元素左移补位。
     * 本方法减少了list中(toIndex - fromIndex)个元素。
     * 若有toIndex==fromIndex，则本方法实际并未对list产生影响。
     */
    protected void removeRange(int fromIndex, int toIndex) {
        // 注2:checkForComodification()
        checkForComodification();
        l.removeRange(fromIndex+offset, toIndex+offset);
        this.modCount = l.modCount;
        size -= (toIndex-fromIndex);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     */
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 后文注释讨论的list均是指视图，其中的index均是视图中的相对值
     * 
     * 将c中所有元素插入list的指定位置(index)，插入顺序为c的迭代器取出的顺序。
     * 原index位置及以后位置的元素顺次向后移动，空出装载c的位置。
     * 
     * 本方法并未定义如下事件发生时的解决策略：
     * 在将c中的元素添加至list的过程中c发生变化
     * 这也意味着如下事件的解决策略同样未定义：将一个非空list添加至自身。
     * 
     * 若list因本方法发生变化则返回true，反之返回false。
     * 
     * list.addAll(set);
     * list.addAll(list.size(), set);
     * 二者等价。
     * 
     * @throws UnsupportedOperationException list不支持本方法。
     * @throws ClassCastException c中任意一个元素因为其所属的类禁止被插入list。
     * @throws NullPointerException c中任意一个元素为null且list禁止包含空元素；或c==null。
     * @throws IllegalArgumentException c中任意一个元素因其某些属性禁止被插入list。
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        // 注1:rangeCheckForAdd(int index)
        rangeCheckForAdd(index);
        int cSize = c.size();
        if (cSize==0)
            return false;

        // 注2:checkForComodification()
        checkForComodification();
        l.addAll(offset+index, c);
        this.modCount = l.modCount;
        size += cSize;
        return true;
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 方法的调用链为：
     * SubList.iterator() ->
     * AbstractList.listIterator() ->
     * SubList.listIterator(final int index)
     * 
     * 若调用SubList.listIterator()
     * 因SubList未重写该方法，因此依然是调用AbstractList.listIterator()
     * 
     * 因此，对于SubList
     * iterator()及listIterator()其实是一回事
     * 最终都是调用SubList.listIterator(final int index)
     */
    public Iterator<E> iterator() {
        return listIterator();
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 后文注释讨论的list均是指视图，其中的index均是视图中的相对值
     * 
     * 返回一个list按固有顺序迭代的listIterator
     * 迭代开始时游标位于索引[index-1,index]之间。
     * 
     * 使用本方法得到listIterator后
     * 若第一次调用的是listIterator.next()，返回的元素的索引是index
     * 同理，若第一次调用的是listIterator.previous()，返回的是索引为index-1的元素。
     * 
     * @throws IndexOutOfBoundsException 索引越界(index < 0 || index > size)。
     */
    public ListIterator<E> listIterator(final int index) {
        // 注2:checkForComodification()
        checkForComodification();
        // 注1:rangeCheckForAdd(int index)
        rangeCheckForAdd(index);

        return new ListIterator<E>() {
            private final ListIterator<E> i = l.listIterator(index+offset);

            public boolean hasNext() {
                return nextIndex() < size;
            }

            public E next() {
                if (hasNext())
                    return i.next();
                else
                    throw new NoSuchElementException();
            }

            public boolean hasPrevious() {
                return previousIndex() >= 0;
            }

            public E previous() {
                if (hasPrevious())
                    return i.previous();
                else
                    throw new NoSuchElementException();
            }

            public int nextIndex() {
                return i.nextIndex() - offset;
            }

            public int previousIndex() {
                return i.previousIndex() - offset;
            }

            public void remove() {
                i.remove();
                SubList.this.modCount = l.modCount;
                size--;
            }

            public void set(E e) {
                i.set(e);
            }

            public void add(E e) {
                i.add(e);
                SubList.this.modCount = l.modCount;
                size++;
            }
        };
    }

    /**
     * 重写父类:java.util.AbstractList<E>
     * 
     * 即视图的视图。
     * 此时视图被看作是原list
     * 但无论如何，最终操作的始终都是最原始的那个list。
     * 一切视图都是对偏移量逻辑的封装
     */
    public List<E> subList(int fromIndex, int toIndex) {
        return new SubList<>(this, fromIndex, toIndex);
    }

    /**
     * 注4:rangeCheck(index)
     */
    private void rangeCheck(int index) {
        if (index < 0 || index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 注1:rangeCheckForAdd(int index)
     */
    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 注2:checkForComodification()
     */
    private void checkForComodification() {
        if (this.modCount != l.modCount)
            throw new ConcurrentModificationException();
    }
}

class RandomAccessSubList<E> extends SubList<E> implements RandomAccess {
    RandomAccessSubList(AbstractList<E> list, int fromIndex, int toIndex) {
        super(list, fromIndex, toIndex);
    }

    /**
     * 重写父类:java.util.SubList<E>
     */
    public List<E> subList(int fromIndex, int toIndex) {
        return new RandomAccessSubList<>(this, fromIndex, toIndex);
    }
}
```

# 已整理层级关系

***本类直接继承的类***

- [java.util.AbstractCollection&lt;E&gt;](/2017/06/19/Java JDK7源码-javautilAbstractCollectionE/)

***本类直接实现的接口***

- [java.util.List&lt;E&gt;](/2017/05/25/Java JDK7源码-javautilListE/)

***直接继承本类的类***

- [java.util.ArrayList&lt;E&gt;](/2017/07/06/Java JDK7源码-javautilArrayListE/)
- [java.util.Vector&lt;E&gt;](/2017/09/05/Java JDK7源码-javautilVectorE/)

# 综述

本类是Java集合框架中的一员，提供了基于支持随机访问的存储结构(形如数组)的List接口的基本实现。

若需实现顺序存储的List(形如链表)，则比起本类，更推荐继承本类的子类，更特化的AbstractSequentialList。

若需实现一个不需改变的List，程序员只需继承本类，并实现get(int index)及size()。

例如，本类某不可变的实现如下(为了填入数据，至少还需要初始化数据的代码，这里放到了构造函数中)：

```
import java.util.AbstractList;

public class UnmoCol<E> extends AbstractList<E> {

    private E[] array;

    public UnmoCol(E[] array) {
        this.array = array;
    }

    @Override
    public E get(int index) {
        return this.array[index];
    }

    @Override
    public int size() {
        return this.array.length;
    }
}
```

调用demo如下：

```
import java.util.AbstractList;
import java.util.Iterator;

public class Main {

    public static void main(String[] args) {
        AbstractList<String> unmoCol = new UnmoCol<String>(new String[]{"reimu", "wang"});
        System.out.println(unmoCol.size());
        for (String s : unmoCol) System.out.println(s);
        Iterator<String> iterator = unmoCol.iterator();
        while (iterator.hasNext()) System.out.println(iterator.next());
    }
}
```

执行本demo输出如下：

```
2
reimu
wang
reimu
wang
```

若需实现一个可变的List，程序员必须额外重写set(int index, E element)，否则会抛出UnsupportedOperationException。若需要List长度可变，程序员必须额外重写add(int index, E element)及remove(int index)。

例如，某个实现如下：

```
import java.util.AbstractList;
import java.util.ArrayList;
import java.util.List;

public class Mocol<E> extends AbstractList<E> {

    private List<E> list = new ArrayList<E>();

    @Override
    public E get(int index) {
        return this.list.get(index);
    }

    @Override
    public int size() {
        return this.list.size();
    }

    @Override
    public void add(int index, E element) {
        this.list.add(index, element);
        this.modCount++;    // 发生结构性变化的次数+1
    }

    @Override
    public E remove(int index) {
        this.modCount++;    // 发生结构性变化的次数+1
        return this.list.remove(index);
    }

    @Override
    public E set(int index, E element) {
        return this.list.set(index, element);
    }
}
```

调用demo如下：

```
import java.util.AbstractList;

public class Main {

    public static void main(String[] args) {
        AbstractList<String> mocol = new Mocol<String>();
        mocol.add("wang");
        mocol.add(0, "reimu");
        for (String str : mocol) System.out.println(str);
        System.out.println(mocol.size());
        mocol.remove(0);
        for (String str : mocol) System.out.println(str);
        mocol.set(0, "change");
        for (String str : mocol) System.out.println(str);
    }
}
```

执行本demo输出如下：

```
reimu
wang
2
wang
change
```

Java集合框架规范建议程序员在继承本类时实现两个构造函数：void(无参数)构造函数；接收一个collection的构造函数。

与其他的Collection接口的抽象实现类不同，程序员在继承本类时不需要提供iterator的实现。本类已实现了iterator及list iterator。

在此之上，程序员需实现或重写那些用于"随机访问"的方法：

```
get(int index)    // 默认abstract
set(int index, E element)    // 默认抛出UnsupportedOperationException
add(int index, E element)    // 默认抛出UnsupportedOperationException
remove(int index)    // 默认抛出UnsupportedOperationException
```

本类所有的非抽象方法都会在文档中有详细的实现描述。这些方法均可被重写。

# 注1:rangeCheckForAdd(int index)

若(index < 0 || index > size())则抛出IndexOutOfBoundsException。

# 注2:checkForComodification()

对于迭代器而言，若modCount != expectedModCount，则认为list存在同步性问题，抛出ConcurrentModificationException。

对于视图而言，若this.modCount != l.modCount，则认为list存在同步性问题，抛出ConcurrentModificationException。

# 注3:Itr.remove()测试

```
import java.util.AbstractList;
import java.util.ArrayList;
import java.util.List;

public class AbstractListTest<E> extends AbstractList<E> {

    private List<E> list = new ArrayList<E>();

    @Override
    public E get(int index) {
        return this.list.get(index);
    }

    @Override
    public int size() {
        return this.list.size();
    }

    @Override
    public void add(int index, E element) {
        this.list.add(index, element);
        this.modCount++;
    }

    @Override
    public E remove(int index) {
        this.modCount++;
        return this.list.remove(index);
    }
}
```

调用：

```
package com.collectiontest.main;

import java.util.AbstractList;
import java.util.Iterator;

public class Main {

    public static void main(String[] args) {
        AbstractList<String> abstractListTest = new AbstractListTest<String>();
        abstractListTest.add("reimu");
        abstractListTest.add("wang");
        Iterator<String> iterator1 = abstractListTest.iterator();
        Iterator<String> iterator2 = abstractListTest.iterator();
        System.out.println(iterator1.next());
        iterator1.remove();
        System.out.println(iterator2.next());
    }
}
```

结果：

```
reimu
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.AbstractList$Itr.checkForComodification(AbstractList.java:386)
	at java.util.AbstractList$Itr.next(AbstractList.java:355)
	at com.collectiontest.main.Main.main(Main.java:16)
```

# 注4:rangeCheck(index)

若index < 0 || index >= size则抛出IndexOutOfBoundsException。

# 未整理层级关系

***直接继承本类的类***

- [java.util.AbstractSequentialList&lt;E&gt;]()