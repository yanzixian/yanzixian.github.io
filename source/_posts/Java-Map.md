---
title: Java-Map
date: 2020-01-15 16:05:49
categories: java
tags: map

---

### 概述

---

此部分[原文](https://www.cnblogs.com/skywang12345/p/3308931.html#a1)

![Map架构.jpg ](Java-Map.assets/Map架构.jpg)

如上图：

- Map 是**映射接口**，Map中存储的内容是**键值对**(key-value)
- `AbstractMap`是**继承于Map的抽象类，它实现了Map中的大部分API**。其它Map的实现类可以通过继承`AbstractMap`来减少重复编码
- `SortedMap` 是继承于Map的接口。`SortedMap`中的内容是**排序的键值对**，排序的方法是通过比较器(Comparator)。
- `NavigableMap` 是继承于`SortedMap`的接口。相比于`SortedMap`，`NavigableMap`有一系列的导航方法；如"获取大于/等于某对象的键值对"、“获取小于/等于某对象的键值对”等等。
- `TreeMap` 继承于`AbstractMap`，且实现了`NavigableMap`接口；因此，`TreeMap`中的内容是“**有序的键值对**”！
- `HashMap` 继承于`AbstractMap`，但没实现`NavigableMap`接口；因此，`HashMap`的内容是“**键值对，但不保证次序**”！
- `Hashtable` 虽然不是继承于`AbstractMap`，但它继承于Dictionary(Dictionary也是键值对的接口)，而且也实现Map接口；因此，`Hashtable`的内容也是“**键值对，也不保证次序**”。但和`HashMap`相比，`Hashtable`是线程安全的，而且它支持通过Enumeration去遍历
- `WeakHashMap` 继承于`AbstractMap`。它和`HashMap`的键类型不同，**`WeakHashMap`的键是“弱键”**。

---

- Map为Java的一个集合接口，位于`java.util`包中
- 采用键-值对的方式存储元素，可以通过键来获取值，键必须唯一不能重复，值可以重复
- Map接口和Collection接口的不同
  - Map是双列的，Collection是单列的
  - Map的键唯一，Collection的子体系Set是唯一的
  - Map集合的数据结构针对键有效，跟值无关；Collection集合的数据结构是针对元素有效

### 功能概述

- 添加功能

  - `V put(K key,V value)`：添加一个键值对，成功返回`null`，失败返回值`value`
  - `void putAll(Map<? extends K,? extends V>)`：向当前Map集合添加另一指定Map集合的所有元素

- 删除功能

  - `void clear()`:移除所有的键值对
  - `V remove(Object key)`:根据值删除键值对，并返回值

- 判断功能

  - `boolean containsKey(Object key)`：判断集合是否包含指定的键
  - `boolean containsValue(Object value)`：判断结合是否包含指定的值
  - `boolean isEmpty()`：判断集合是否为空
  - `boolean equals(Object obj)`：比较指定的对象（通常也是一个Map集合）与此映射是否相等（拥有相同的键值对）

- 获取功能

  - `Set<Map.Entry<K,V>> entrySet()`：返回键值对的`Set`集合
  - `V get(Object key)`：根据键获取值
  - `Set keySet()`：获取集合中所有键的集合
  - `Collection<V> values()`：获取集合中所有值的集合
  - `int size()`：返回集合中键值对的个数

- 其他

  - `int hashCode()`：返回map集合的hash值

    ```java
    Map map=new HashMap();
    int hashCode=map.hashCode();
    System.out.println("map集合没有元素时的hash值为"+hashcode);
    map.put("k1","v1");
    hashcode=map.hashcode();
    System.out.println("map集合有元素时的hash值为"+hashcode);
    ```

---

下面一部分基于[原文](https://blog.csdn.net/kyi_zhu123/article/details/52769469)

`Map.entrySet()` 这个方法返回的是一个`Set<Map.Entry<K,V>>`，`Map.Entry` 是Map中的一个接口，他的用途是表示一个映射项（里面有Key和Value），而`Set<Map.Entry<K,V>>`表示一个映射项的Set。`Map.Entry`里有相应的`getKey`和`getValue`方法，即`JavaBean`，让我们能够从一个项中取出Key和Value。

遍历Map的四种方法：

```java
public static void main(String[] args) {
 
 
  Map<String, String> map = new HashMap<String, String>();
  map.put("1", "value1");
  map.put("2", "value2");
  map.put("3", "value3");
  
  //第一种：普遍使用，二次取值
  System.out.println("通过Map.keySet遍历key和value：");
  for (String key : map.keySet()) {
   System.out.println("key= "+ key + " and value= " + map.get(key));
  }
  
  //第二种
  System.out.println("通过Map.entrySet使用iterator遍历key和value：");
  Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();
  while (it.hasNext()) {
   Map.Entry<String, String> entry = it.next();
   System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
  }
  
  //第三种：推荐，尤其是容量大时
  System.out.println("通过Map.entrySet遍历key和value");
  for (Map.Entry<String, String> entry : map.entrySet()) {
   System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
  }
 
  //第四种
  System.out.println("通过Map.values()遍历所有的value，但不能遍历key");
  for (String v : map.values()) {
   System.out.println("value= " + v);
  }
 }

```

---

此部分基于[此文](https://www.cnblogs.com/skywang12345/p/3308931.html#a1)，加上一些自己的理解和补充

### Map

**定义：**`public interface Map<K,V> {}`

Map接口提供三种collection视图，允许以**键集**、**值集**或**键-值映**射关系集的形式查看某个映射的内容

Map 的实现类应该提供2个“标准的”构造方法：

- **第一个，void（无参数）构造方法，用于创建空映射**；

- **第二个，带有单个 Map 类型参数的构造方法，用于创建一个与其参数具有相同键-值映射关系的新映射。**

**API：**

```java
abstract void                 clear()
abstract boolean              containsKey(Object key)
abstract boolean              containsValue(Object value)
abstract Set<Entry<K, V>>     entrySet()//返回键-值对的set集合
abstract boolean              equals(Object object)
abstract V                    get(Object key)
abstract int                  hashCode()
abstract boolean              isEmpty()
abstract Set<K>               keySet()//返回键的set集合
abstract V                    put(K key, V value)
abstract void                 putAll(Map<? extends K, ? extends V> map)
abstract V                    remove(Object key)
abstract int                  size()
abstract Collection<V>        values()//返回值的collection集合，值可能重复
```

#### `Map.Entry`

**定义：**`interface Entry<K,V> {}`

`Map.Entry`是Map中内部的一个接口，`Map.Entry`是**键值对**，Map通过 `entrySet`() 获取`Map.Entry`的键值对集合，从而通过该集合实现对键值对的操作。

**`Map.Entry`的API**

```java
abstract boolean     equals(Object object)
abstract K           getKey()
abstract V           getValue()
abstract int         hashCode()
abstract V           setValue(V object)
```

### AbstractMap

**定义：**`public abstract class AbstractMap<K,V> implements Map<K,V> {}`

`AbstractMap`类提供 Map 接口的骨干实现，以最大限度地减少实现此接口所需的工作。

要实现不可修改的映射，编程人员只需扩展此类并提供 entrySet 方法的实现即可，该方法将返回映射的映射关系 set 视图。通常，返回的 set 将依次在 `AbstractSet` 上实现。此 set 不支持 add() 或 remove() 方法，其迭代器也不支持 remove() 方法。

要实现可修改的映射，编程人员必须另外重写此类的 put 方法（否则将抛出 `UnsupportedOperationException`），`entrySet().iterator()` 返回的迭代器也必须另外实现其 remove 方法。

**AbstractMap的API**

```java
abstract Set<Entry<K, V>>     entrySet()//没有实现此方法
         void                 clear()
         boolean              containsKey(Object key)
         boolean              containsValue(Object value)
         boolean              equals(Object object)
         V                    get(Object key)
         int                  hashCode()
         boolean              isEmpty()
         Set<K>               keySet()
         V                    put(K key, V value)
         void                 putAll(Map<? extends K, ? extends V> map)
         V                    remove(Object key)
         int                  size()
         String               toString()
         Collection<V>        values()
         Object               clone()
```

### SortedMap

**定义：**`public interface SortedMap<K,V> extends Map<K,V> { }`

SortedMap是一个继承于Map接口的接口。它是一个有序的SortedMap键值映射。
SortedMap的排序方式有两种：**自然排序** 或者 **用户指定比较器**。 插入有序 SortedMap 的所有元素都必须实现 Comparable 接口（或者被指定的比较器所接受）

另外，所有SortedMap 实现类都应该提供 **4 个“标准”构造方法：**

- **void（无参数）构造方法**，它创建一个空的有序映射，按照键的自然顺序进行排序
- **带有一个 Comparator 类型参数的构造方法**，它创建一个空的有序映射，根据指定的比较器进行排序
- **带有一个 Map 类型参数的构造方法**，它创建一个新的有序映射，其键-值映射关系与参数相同，按照键的自然顺序进行排序
- **带有一个 SortedMap 类型参数的构造方法**，它创建一个新的有序映射，其键-值映射关系和排序方法与输入的有序映射相同。无法保证强制实施此建议，因为接口不能包含构造方法

**SortedMap的API**

```java
// 继承于Map的API
abstract void                 clear()
abstract boolean              containsKey(Object key)
abstract boolean              containsValue(Object value)
abstract Set<Entry<K, V>>     entrySet()
abstract boolean              equals(Object object)
abstract V                    get(Object key)
abstract int                  hashCode()
abstract boolean              isEmpty()
abstract Set<K>               keySet()
abstract V                    put(K key, V value)
abstract void                 putAll(Map<? extends K, ? extends V> map)
abstract V                    remove(Object key)
abstract int                  size()
abstract Collection<V>        values()
    
// SortedMap新增的API 
abstract Comparator<? super K>     comparator()
abstract K                         firstKey()
abstract SortedMap<K, V>           headMap(K endKey)
abstract K                         lastKey()
abstract SortedMap<K, V>           subMap(K startKey, K endKey)
abstract SortedMap<K, V>           tailMap(K startKey)
```

### NavigableMap

**定义**：`public interface NavigableMap<K,V> extends SortedMap<K,V> { }`

NavigableMap是继承于SortedMap的接口。它是一个可导航的键-值对集合，具有了为给定搜索目标报告最接近匹配项的导航方法。
NavigableMap分别提供了获取“键”、“键-值对”、“键集”、“键-值对集”的相关方法

**NavigableMap的API**

```java
abstract Entry<K, V>             ceilingEntry(K key)
abstract Entry<K, V>             firstEntry()
abstract Entry<K, V>             floorEntry(K key)
abstract Entry<K, V>             higherEntry(K key)
abstract Entry<K, V>             lastEntry()
abstract Entry<K, V>             lowerEntry(K key)
abstract Entry<K, V>             pollFirstEntry()
abstract Entry<K, V>             pollLastEntry()
abstract K                       ceilingKey(K key)
abstract K                       floorKey(K key)
abstract K                       higherKey(K key)
abstract K                       lowerKey(K key)
abstract NavigableSet<K>         descendingKeySet()
abstract NavigableSet<K>         navigableKeySet()
abstract NavigableMap<K, V>      descendingMap()
abstract NavigableMap<K, V>      headMap(K toKey, boolean inclusive)
abstract SortedMap<K, V>         headMap(K toKey)
abstract SortedMap<K, V>         subMap(K fromKey, K toKey)
abstract NavigableMap<K, V>      subMap(K fromKey, boolean fromInclusive, K toKey, boolean toInclusive)
abstract SortedMap<K, V>         tailMap(K fromKey)
abstract NavigableMap<K, V>      tailMap(K fromKey, boolean inclusive)
```

注：

NavigableMap除了继承SortedMap的特性外，它的提供的功能可以分为4类：

- 第1类，**提供操作键-值对的方法。**
   lowerEntry、floorEntry、ceilingEntry 和 higherEntry 方法，它们分别返回与小于、小于等于、大于等于、大于给定键的键关联的 Map.Entry 对象。firstEntry、pollFirstEntry、lastEntry 和 pollLastEntry 方法，它们返回和/或移除最小和最大的映射关系（如果存在），否则返回 null
- 第2类，**提供操作键的方法**。这个和第1类比较类似lowerKey，floorKey、ceilingKey 和 higherKey 方法，它们分别返回与小于、小于等于、大于等于、大于给定键的键
- 第3类，**获取键集。**navigableKeySet、descendingKeySet分别获取正序/反序的键集
- 第4类，获取键-值对的子集。

### Dictionary

**定义：**`public abstract class Dictionary<K,V> {}`

NavigableMap是JDK 1.0定义的键值对的接口，它也包括了操作键值对的基本函数。

**Dictionary的API**

```java
abstract Enumeration<V>     elements()
abstract V                  get(Object key)
abstract boolean            isEmpty()
abstract Enumeration<K>     keys()
abstract V                  put(K key, V value)
abstract V                  remove(Object key)
abstract int                size()
```

---

