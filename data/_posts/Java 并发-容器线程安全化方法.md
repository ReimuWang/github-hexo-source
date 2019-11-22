---
title: Java 并发-容器线程安全化方法
date: 2018-02-01 17:20:36
tags: [Java,并发]
categories: Java 并发
---

Java API中提供了常见的线程安全的容器，可参见[Java 并发-线程安全的容器](/2018/01/24/Java 并发-线程安全的容器/)。这些容器专为并发环境设计，性能极高。但是在串-并行混合，对性能要求又不高时，比起专门声明新的线程安全的容器，我们更希望能有一个简单的方式将线程不安全的容器改造为线程安全的。这部分功能被封装在java.util.Collections工具类的容器线程安全化方法中：这些方法接收一个特定的容器，然后将它改造为线程安全的容器后返回:

```
public static <T> Collection<T> synchronizedCollection(Collection<T> c)

public static <T> List<T> synchronizedList(List<T> list)

public static <T> Set<T> synchronizedSet(Set<T> s)

public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m)
```

<!-- more -->

# synchronizedCollection

全部代码如下：

```
public static <T> Collection<T> synchronizedCollection(Collection<T> c) {
    return new SynchronizedCollection<>(c);
}
```

它的返回值是Collections的静态成员内部类，全部代码如下：

```
static class SynchronizedCollection<E> implements Collection<E>, Serializable {
    private static final long serialVersionUID = 3053995032091335093L;

    final Collection<E> c;
    final Object mutex;

    SynchronizedCollection(Collection<E> c) {
        if (c==null)
            throw new NullPointerException();
        this.c = c;
        mutex = this;
    }
    SynchronizedCollection(Collection<E> c, Object mutex) {
        this.c = c;
        this.mutex = mutex;
    }

    public int size() {
        synchronized (mutex) {return c.size();}
    }
    public boolean isEmpty() {
        synchronized (mutex) {return c.isEmpty();}
    }
    public boolean contains(Object o) {
        synchronized (mutex) {return c.contains(o);}
    }
    public Object[] toArray() {
        synchronized (mutex) {return c.toArray();}
    }
    public <T> T[] toArray(T[] a) {
        synchronized (mutex) {return c.toArray(a);}
    }

    public Iterator<E> iterator() {
        return c.iterator();
    }

    public boolean add(E e) {
        synchronized (mutex) {return c.add(e);}
    }
    public boolean remove(Object o) {
        synchronized (mutex) {return c.remove(o);}
    }

    public boolean containsAll(Collection<?> coll) {
        synchronized (mutex) {return c.containsAll(coll);}
    }
    public boolean addAll(Collection<? extends E> coll) {
        synchronized (mutex) {return c.addAll(coll);}
    }
    public boolean removeAll(Collection<?> coll) {
        synchronized (mutex) {return c.removeAll(coll);}
    }
    public boolean retainAll(Collection<?> coll) {
        synchronized (mutex) {return c.retainAll(coll);}
    }
    public void clear() {
        synchronized (mutex) {c.clear();}
    }
    public String toString() {
        synchronized (mutex) {return c.toString();}
    }
    private void writeObject(ObjectOutputStream s) throws IOException {
        synchronized (mutex) {s.defaultWriteObject();}
    }
}
```

原来如此，SynchronizedCollection以mutex为监视器，在c的方法外部都包装上了一层synchronized。具体的业务逻辑依然是调用c的方法，确实相当于将原容器线程安全化了。不过本质上来说，这和在容器的外部代码中使用synchronized是一回事，性能不高也是理所当然的了。

SynchronizedCollection共有两个构造函数，其区别就在于是否指定mutex。Collections.SynchronizedCollection()并未传入监视器对象，此时监视器对象将为SynchronizedCollection实例本身。

那么指定mutex的那个构造函数又有什么用处呢？Collections有如下方法：

```
static <T> Collection<T> synchronizedCollection(Collection<T> c, Object mutex) {
    return new SynchronizedCollection<>(c, mutex);
}
```

该方法供java.util包内部调用，并非暴露出去的公共方法，故在此就不深究了。

# synchronizedList

套路与Collections.SynchronizedCollection()相同，因此只贴出相关代码，就不再赘述细节了：

```
public static <T> List<T> synchronizedList(List<T> list) {
    return (list instanceof RandomAccess ?
            new SynchronizedRandomAccessList<>(list) :
            new SynchronizedList<>(list));
}

static <T> List<T> synchronizedList(List<T> list, Object mutex) {
    return (list instanceof RandomAccess ?
            new SynchronizedRandomAccessList<>(list, mutex) :
            new SynchronizedList<>(list, mutex));
}

static class SynchronizedList<E> extends SynchronizedCollection<E> implements List<E> {
    private static final long serialVersionUID = -7754090372962971524L;

    final List<E> list;

    SynchronizedList(List<E> list) {
        super(list);
        this.list = list;
    }
    SynchronizedList(List<E> list, Object mutex) {
        super(list, mutex);
        this.list = list;
    }

    public boolean equals(Object o) {
        if (this == o)
            return true;
        synchronized (mutex) {return list.equals(o);}
    }
    public int hashCode() {
        synchronized (mutex) {return list.hashCode();}
    }

    public E get(int index) {
        synchronized (mutex) {return list.get(index);}
    }
    public E set(int index, E element) {
        synchronized (mutex) {return list.set(index, element);}
    }
    public void add(int index, E element) {
        synchronized (mutex) {list.add(index, element);}
    }
    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);}
    }

    public int indexOf(Object o) {
        synchronized (mutex) {return list.indexOf(o);}
    }
    public int lastIndexOf(Object o) {
        synchronized (mutex) {return list.lastIndexOf(o);}
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        synchronized (mutex) {return list.addAll(index, c);}
    }

    public ListIterator<E> listIterator() {
        return list.listIterator();
    }

    public ListIterator<E> listIterator(int index) {
        return list.listIterator(index);
    }

    public List<E> subList(int fromIndex, int toIndex) {
        synchronized (mutex) {
            return new SynchronizedList<>(list.subList(fromIndex, toIndex),
                                        mutex);
        }
    }

    private Object readResolve() {
        return (list instanceof RandomAccess
                ? new SynchronizedRandomAccessList<>(list)
                : this);
    }
}

static class SynchronizedRandomAccessList<E> extends SynchronizedList<E> implements RandomAccess {

    SynchronizedRandomAccessList(List<E> list) {
        super(list);
    }

    SynchronizedRandomAccessList(List<E> list, Object mutex) {
        super(list, mutex);
    }

    public List<E> subList(int fromIndex, int toIndex) {
        synchronized (mutex) {
            return new SynchronizedRandomAccessList<>(
                list.subList(fromIndex, toIndex), mutex);
        }
    }

    private static final long serialVersionUID = 1530674583602358482L;

    private Object writeReplace() {
        return new SynchronizedList<>(list);
    }
}
```

# synchronizedSet

同synchronizedList，只贴出代码：

```
public static <T> Set<T> synchronizedSet(Set<T> s) {
    return new SynchronizedSet<>(s);
}

static <T> Set<T> synchronizedSet(Set<T> s, Object mutex) {
    return new SynchronizedSet<>(s, mutex);
}

static class SynchronizedSet<E> extends SynchronizedCollection<E> implements Set<E> {
    private static final long serialVersionUID = 487447009682186044L;

    SynchronizedSet(Set<E> s) {
        super(s);
    }
    SynchronizedSet(Set<E> s, Object mutex) {
        super(s, mutex);
    }

    public boolean equals(Object o) {
        if (this == o)
            return true;
        synchronized (mutex) {return c.equals(o);}
    }
    public int hashCode() {
        synchronized (mutex) {return c.hashCode();}
    }
}
```

# synchronizedMap

这个就稍稍有些特殊了，不过我们还是先贴出代码：

```
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
    return new SynchronizedMap<>(m);
}

private static class SynchronizedMap<K,V> implements Map<K,V>, Serializable {
    private static final long serialVersionUID = 1978198479659022715L;

    private final Map<K,V> m;
    final Object      mutex;

    SynchronizedMap(Map<K,V> m) {
        if (m==null)
            throw new NullPointerException();
        this.m = m;
        mutex = this;
    }

    SynchronizedMap(Map<K,V> m, Object mutex) {
        this.m = m;
        this.mutex = mutex;
    }

    public int size() {
        synchronized (mutex) {return m.size();}
    }
    public boolean isEmpty() {
        synchronized (mutex) {return m.isEmpty();}
    }
    public boolean containsKey(Object key) {
        synchronized (mutex) {return m.containsKey(key);}
    }
    public boolean containsValue(Object value) {
        synchronized (mutex) {return m.containsValue(value);}
    }
    public V get(Object key) {
        synchronized (mutex) {return m.get(key);}
    }

    public V put(K key, V value) {
        synchronized (mutex) {return m.put(key, value);}
    }
    public V remove(Object key) {
        synchronized (mutex) {return m.remove(key);}
    }
    public void putAll(Map<? extends K, ? extends V> map) {
        synchronized (mutex) {m.putAll(map);}
    }
    public void clear() {
        synchronized (mutex) {m.clear();}
    }

    private transient Set<K> keySet = null;
    private transient Set<Map.Entry<K,V>> entrySet = null;
    private transient Collection<V> values = null;

    public Set<K> keySet() {
        synchronized (mutex) {
            if (keySet==null)
                keySet = new SynchronizedSet<>(m.keySet(), mutex);
            return keySet;
        }
    }

    public Set<Map.Entry<K,V>> entrySet() {
        synchronized (mutex) {
            if (entrySet==null)
                entrySet = new SynchronizedSet<>(m.entrySet(), mutex);
            return entrySet;
        }
    }

    public Collection<V> values() {
        synchronized (mutex) {
            if (values==null)
                values = new SynchronizedCollection<>(m.values(), mutex);
            return values;
        }
    }

    public boolean equals(Object o) {
        if (this == o)
            return true;
        synchronized (mutex) {return m.equals(o);}
    }
    public int hashCode() {
        synchronized (mutex) {return m.hashCode();}
    }
    public String toString() {
        synchronized (mutex) {return m.toString();}
    }
    private void writeObject(ObjectOutputStream s) throws IOException {
        synchronized (mutex) {s.defaultWriteObject();}
    }
}
```

之所以说它有些特殊，是因为Map的线程安全化方法被放在Collections类中是略有不妥的，估计是Java API的设计人员想将所有的容器线程安全化方法集中到一个类中吧。正因为如此，SynchronizedMap的访问权限是私有，且没有传入监视器对象的Collections.synchronizedMap()。