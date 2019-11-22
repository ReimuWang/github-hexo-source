---
title: Java JDK7源码-java.util.AbstractMap&lt;K,V&gt;
date: 2018-11-09 11:28:36
tags: [Java,源码]
categories: Java JDK7源码
---

# 源码

<!-- more -->

```
package java.util;
import java.util.Map.Entry;

public abstract class AbstractMap<K,V> implements Map<K,V> {

    protected AbstractMap() {
    }

    // 查询操作

    /**
     * 实现接口:java.util.Map<K,V>
     */
    public int size() {
        return entrySet().size();
    }

    /**
     * 实现接口:java.util.Map<K,V>
     */
    public boolean isEmpty() {
        return size() == 0;
    }

    /**
     * 实现接口:java.util.Map<K,V>
     *
     * 本实现迭代entrySet()返回的set，搜寻值为value的entry
     * 若找到，则返回true
     * 若直到迭代结束也未找到，则返回false
     * 
     * 需要注意的是，本实现的时间复杂度为线性阶(基于map的大小)
     *
     * @throws ClassCastException value的类型与map不合
     * @throws NullPointerException value==null且map禁止包含null
     */
    public boolean containsValue(Object value) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (value==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getValue()==null)
                    return true;
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (value.equals(e.getValue()))
                    return true;
            }
        }
        return false;
    }

    /**
     * 实现接口:java.util.Map<K,V>
     * 
     * 本实现迭代entrySet()返回的set，搜寻键为key的entry
     * 若找到，则返回true
     * 若直到迭代结束也未找到，则返回false
     * 
     * 需要注意的是，本实现的时间复杂度为线性阶(基于map的大小)
     * 许多实现均会重写本方法
     *
     * @throws ClassCastException key的类型与map不合
     * @throws NullPointerException key==null且map禁止包含null
     */
    public boolean containsKey(Object key) {
        Iterator<Map.Entry<K,V>> i = entrySet().iterator();
        if (key==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    return true;
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    return true;
            }
        }
        return false;
    }

    /**
     * 实现接口:java.util.Map<K,V>
     * 
     * 本实现迭代entrySet()返回的set，搜寻键为key的entry
     * 若找到，则返回entry的值
     * 若直到迭代结束也未找到，则返回null
     *
     * 需要注意的是，本实现的时间复杂度为线性阶(基于map的大小)
     * 许多实现均会重写本方法
     *
     * @throws ClassCastException key的类型与map不合
     * @throws NullPointerException key==null且map禁止包含null
     */
    public V get(Object key) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (key==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    return e.getValue();
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    return e.getValue();
            }
        }
        return null;
    }


    // 修改操作

    /**
     * 实现接口:java.util.Map<K,V>
     * 
     * 本实现总是会抛出UnsupportedOperationException
     *
     * @throws UnsupportedOperationException map不支持本方法
     * @throws ClassCastException key或value因其类型禁止被插入map
     * @throws NullPointerException key为null且map的键不允许为null 或 value为null且map的值不允许为null
     * @throws IllegalArgumentException key或value因其某些属性禁止被插入map
     */
    public V put(K key, V value) {
        throw new UnsupportedOperationException();
    }

    /**
     * 实现接口:java.util.Map<K,V>
     *
     * 本实现迭代entrySet()返回的set，搜寻键为key的entry
     * 若找到，则先使用entry.getValue方法得到它的值
     * 然后使用iterator的remove操作将entry自set中移除(当然，map中对应的键值对也被移除了)
     * 最后返回此前保留的值
     * 若直到迭代结束也未找到，则返回null
     * 
     * 需要注意的是，本实现的时间复杂度为线性阶(基于map的大小)
     * 许多实现均会重写本方法
     * 
     * 注意：若map包含key，且entrySet方法返回的set的iterator不支持remove方法
     * 则抛出UnsupportedOperationException
     *
     * @throws UnsupportedOperationException map不支持本方法
     * @throws ClassCastException key的类型与map不合
     * @throws NullPointerException key为null且map的键不允许为null
     */
    public V remove(Object key) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        Entry<K,V> correctEntry = null;
        if (key==null) {
            while (correctEntry==null && i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    correctEntry = e;
            }
        } else {
            while (correctEntry==null && i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    correctEntry = e;
            }
        }

        V oldValue = null;
        if (correctEntry !=null) {
            oldValue = correctEntry.getValue();
            i.remove();
        }
        return oldValue;
    }


    // 批量操作

    /**
     * 实现接口:java.util.Map<K,V>
     * 
     * 本实现迭代m.entrySet()返回的set
     * 然后针对每次迭代的结果，均将键值对使用map.put方法存入map中
     * 
     * 注意：若map不支持put操作，且m!=null
     * 则抛出UnsupportedOperationException
     *
     * @throws UnsupportedOperationException map不支持本方法
     * @throws ClassCastException m中的某个key或value因其类型禁止被插入map
     * @throws NullPointerException m==null 或m中存在为null的key且map的键不允许为null 或 m中存在为null的value且map的值不允许为null
     * @throws IllegalArgumentException m中的某个key或value因其某些属性禁止被插入map
     */
    public void putAll(Map<? extends K, ? extends V> m) {
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            put(e.getKey(), e.getValue());
    }

    /**
     * 实现接口:java.util.Map<K,V>
     * 
     * 注意：若entrySet()返回的set不支持clear操作
     * 则抛出UnsupportedOperationException
     *
     * @throws UnsupportedOperationException map不支持本方法
     */
    public void clear() {
        entrySet().clear();
    }


    // 视图

    /**
     * keySet，values会在第一次使用时初始化
     * 包含key或value的容器视图均只需要一份
     */
    transient volatile Set<K>        keySet = null;
    transient volatile Collection<V> values = null;

    /**
     * 实现接口:java.util.Map<K,V>
     * 
     * 本实现返回一个set，它是java.util.AbstractSet的子类
     * 该set的iterator是map.entrySet()的iterator的"包装对象"
     * 该set的size方法基于map.size()
     * 该set的contains(Object o)方法基于map.containsKey方法
     * 
     * 本方法所返回的set会在方法第一次被调用时创建
     * 随后再调用本方法时，均会返回同一个set
     * 
     * 本方法并未进行并发控制
     * 因此在并发环境下，如果复数个请求同时调用本方法
     * 并不能确保返回的set是唯一的
     */
    public Set<K> keySet() {
        if (keySet == null) {
            keySet = new AbstractSet<K>() {
                public Iterator<K> iterator() {
                    return new Iterator<K>() {
                        private Iterator<Entry<K,V>> i = entrySet().iterator();

                        public boolean hasNext() {
                            return i.hasNext();
                        }

                        public K next() {
                            return i.next().getKey();
                        }

                        public void remove() {
                            i.remove();
                        }
                    };
                }

                public int size() {
                    return AbstractMap.this.size();
                }

                public boolean isEmpty() {
                    return AbstractMap.this.isEmpty();
                }

                public void clear() {
                    AbstractMap.this.clear();
                }

                public boolean contains(Object k) {
                    return AbstractMap.this.containsKey(k);
                }
            };
        }
        return keySet;
    }

    /**
     * 实现接口:java.util.Map<K,V>
     * 
     * 本实现返回一个collection，它是java.util.AbstractCollection的子类
     * 该collection的iterator是map.entrySet()的iterator的"包装对象"
     * 该collection的size方法基于map.size()
     * 该collection的contains(Object o)方法基于map.containsValue方法
     * 
     * 本方法所返回的collection会在方法第一次被调用时创建
     * 随后再调用本方法时，均会返回同一个collection
     * 
     * 本方法并未进行并发控制
     * 因此在并发环境下，如果复数个请求同时调用本方法
     * 并不能确保返回的collection是唯一的
     */
    public Collection<V> values() {
        if (values == null) {
            values = new AbstractCollection<V>() {
                public Iterator<V> iterator() {
                    return new Iterator<V>() {
                        private Iterator<Entry<K,V>> i = entrySet().iterator();

                        public boolean hasNext() {
                            return i.hasNext();
                        }

                        public V next() {
                            return i.next().getValue();
                        }

                        public void remove() {
                            i.remove();
                        }
                    };
                }

                public int size() {
                    return AbstractMap.this.size();
                }

                public boolean isEmpty() {
                    return AbstractMap.this.isEmpty();
                }

                public void clear() {
                    AbstractMap.this.clear();
                }

                public boolean contains(Object v) {
                    return AbstractMap.this.containsValue(v);
                }
            };
        }
        return values;
    }

    public abstract Set<Entry<K,V>> entrySet();


    // 比较与哈希

    /**
     * 重写祖先类:java.lang.Object
     * 实现接口:java.util.Map<K,V>
     * 
     * 比较o与map的相等性
     * 若o同样是一个Map且o与map中存储的键值对均相等，则返回true
     * 更正式的说，如果满足如下条件，则可认为两个Map m1 m2相等：
     * m1.entrySet().equals(m2.entrySet())
     * 以这种方式设计的话，即便m1 m2的实现类不同，也可以正确的判断二者是否相等
     * 
     * 本实现的判断规则为：
     * 1.如果o是map本身，返回true
     * 2.如果o的类型不是Map，返回false
     * 3.如果o的长度与map不相等，返回false
     * 4.迭代map.entrySet方法返回的set，检查o是否包含所有的迭代出的entry，若存在不包含的情况，返回false
     * 5.若直至迭代结束也未找到不包含的情况，则返回true
     */
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<K,V> m = (Map<K,V>) o;
        if (m.size() != size())
            return false;

        try {
            Iterator<Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(m.get(key)==null && m.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(m.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused) {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true;
    }

    /**
     * 重写祖先类:java.lang.Object
     * 实现接口:java.util.Map<K,V>
     * 
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
    public int hashCode() {
        int h = 0;
        Iterator<Entry<K,V>> i = entrySet().iterator();
        while (i.hasNext())
            h += i.next().hashCode();
        return h;
    }

    /**
     * 重写祖先类:java.lang.Object
     * 
     * 返回描述map的字符串
     * 该字符串最外侧是一组大括号{}
     * 其内部是由map中的键值对。顺序为map的entrySet的iterator返回的顺序
     * 相邻的键值对间以", "(逗号+空格)分割
     * 每组键值对的形式为：
     * key=value
     * key与value均会调用String类的valueOf(Object obj)方法转换为字符串
     */
    public String toString() {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (! i.hasNext())
            return "{}";

        StringBuilder sb = new StringBuilder();
        sb.append('{');
        for (;;) {
            Entry<K,V> e = i.next();
            K key = e.getKey();
            V value = e.getValue();
            sb.append(key   == this ? "(this Map)" : key);
            sb.append('=');
            sb.append(value == this ? "(this Map)" : value);
            if (! i.hasNext())
                return sb.append('}').toString();
            sb.append(',').append(' ');
        }
    }

    /**
     * 重写祖先类:java.lang.Object
     * 
     * 返回map的浅拷贝
     * 即不会复制key与value本身
     */
    protected Object clone() throws CloneNotSupportedException {
        AbstractMap<K,V> result = (AbstractMap<K,V>)super.clone();
        result.keySet = null;
        result.values = null;
        return result;
    }

    /**
     * SimpleEntry与SimpleImmutableEntry均会调用的通用方法
     * 测试相等性，检查null
     */
    private static boolean eq(Object o1, Object o2) {
        return o1 == null ? o2 == null : o1.equals(o2);
    }

    // 注意：
    // 即便共享部分代码
    // 但SimpleEntry与SimpleImmutableEntry是明确不同的两个类
    // 不过共用的代码不多
    // 因此没必要再弄出个抽象类

    /**
     * Entry中存储着Map的键值对
     * 调用entry.setValue方法可以改变value
     * 本类用于辅助AbstractMap的子类构建map
     * (继承AbstractMap，构建map，最重要的一环就是entrySet方法的编写)
     */
    public static class SimpleEntry<K,V>
        implements Entry<K,V>, java.io.Serializable
    {
        private static final long serialVersionUID = -8499721149061103585L;

        private final K key;
        private V value;

        /**
         * 以key-value为键值对创建entry
         */
        public SimpleEntry(K key, V value) {
            this.key   = key;
            this.value = value;
        }

        /**
         * 使用传入entry的键值对创建新的entry
         */
        public SimpleEntry(Entry<? extends K, ? extends V> entry) {
            this.key   = entry.getKey();
            this.value = entry.getValue();
        }

        /**
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         */
        public K getKey() {
            return key;
        }

        /**
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         */
        public V getValue() {
            return value;
        }

        /**
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         */
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }

        /**
         * 重写祖先类:java.lang.Object
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         * 
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
        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            return eq(key, e.getKey()) && eq(value, e.getValue());
        }

        /**
         * 重写祖先类:java.lang.Object
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         * 
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
        public int hashCode() {
            return (key   == null ? 0 :   key.hashCode()) ^
                   (value == null ? 0 : value.hashCode());
        }

        /**
         * 重写祖先类:java.lang.Object
         * 
         * 返回描述entry的字符串
         * 本实现返回：
         * key=value
         */
        public String toString() {
            return key + "=" + value;
        }

    }

    /**
     * Entry中存储着Map的键值对，该键值对不可变
     * 本类不支持entry.setValue方法
     * 本类用于辅助AbstractMap的子类构建map，可以方便的返回线程安全的键值对快照
     * (继承AbstractMap，构建map，最重要的一环就是entrySet方法的编写)
     */
    public static class SimpleImmutableEntry<K,V>
        implements Entry<K,V>, java.io.Serializable
    {
        private static final long serialVersionUID = 7138329143949025153L;

        private final K key;
        private final V value;

        /**
         * 以key-value为键值对创建entry
         */
        public SimpleImmutableEntry(K key, V value) {
            this.key   = key;
            this.value = value;
        }

        /**
         * 使用传入entry的键值对创建新的entry
         */
        public SimpleImmutableEntry(Entry<? extends K, ? extends V> entry) {
            this.key   = entry.getKey();
            this.value = entry.getValue();
        }

        /**
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         */
        public K getKey() {
            return key;
        }

        /**
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         */
        public V getValue() {
            return value;
        }

        /**
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         * 
         * 因SimpleImmutableEntry不可变
         * 则总是抛出UnsupportedOperationException
         *
         * @throws UnsupportedOperationException 总是抛出该异常
         */
        public V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        /**
         * 重写祖先类:java.lang.Object
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         * 
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
        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            return eq(key, e.getKey()) && eq(value, e.getValue());
        }

        /**
         * 重写祖先类:java.lang.Object
         * 实现接口:java.util.Map<K,V>.Entry<K,V>
         * 
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
        public int hashCode() {
            return (key   == null ? 0 :   key.hashCode()) ^
                   (value == null ? 0 : value.hashCode());
        }

        /**
         * 重写祖先类:java.lang.Object
         * 
         * 返回描述entry的字符串
         * 本实现返回：
         * key=value
         */
        public String toString() {
            return key + "=" + value;
        }

    }

}
```

# 已整理层级关系

***直接继承本类的类***

- [java.util.HashMap&lt;K,V&gt;](/2018/11/09/Java JDK7源码-javautilHashMapKV/)

***本类直接实现的接口***

- [java.util.Map&lt;K,V&gt;](/2018/11/08/Java JDK7源码-javautilMapKV/)

# 综述

本类是Java集合框架中的一员。其中K，V分别是map中key与value的类型。提供了Map接口最小化，也是最基本的实现。

若需实现一个不可变的map，程序员只需继承本类，然后实现entrySet方法，该方法会返回map键值对的set视图。该set应基于java.util.AbstractSet进行设计，不能支持add或remove方法，它的iterator不能支持remove方法。

如需实现可变的map，在此基础上程序员还需实现本类的put方法(本类会抛出UnsupportedOperationException异常)，同时entrySet().iterator()返回的iterator还需实现remove方法。

遵循Map接口的规范，通常来说，程序员需要实现两个构造函数：

1. 创建一个空的map的无参构造函数
2. 接收一个Map类型参数的构造函数，它会以入参为基础，创建一个类型为自身实现类的，由相同键值对(key-value)构成的新map

对于本类的非抽象方法，如果实现类认为有必要，均可以重写以达到更高的性能标准。