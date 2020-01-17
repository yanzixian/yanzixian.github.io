---
title: Java-Map
date: 2020-01-15 16:05:49
categories: java
tags: map
---

### 概述

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

