---
title: 'Java-Collections,Arrays工具类'
date: 2020-01-21 11:00:38
categories: java
tags: 
---

### 概述

Collections，Arrays是两个位于java.util包中中的工具类，主要提供对于集合和数组的操作方法，其提供的方法均是静态方法，无需实例化对象，通过类名直接调用即可

### Collections

#### 定义：

`public class Collections {...}`

与Collection不同，Collection是一个集合接口，其中的方法只进行定义，没有实现；而Collections是一个类，其中的方法均进行了实现

#### 提供的方法：

```java
public static <T extends Comparable<? super T>> void sort(List<T> list){...}//对指定列表中的元素进行升序排列，列表中的元素必须实现Comparable接口，排序算法稳定
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {...}//在指定的列表中使用二分查找搜索指定的元素，列表中元素必须为升序排列；若列表中有多个与key相同的元素，不能保证找到哪一个;主要实现是根据列表的大小和是否可以随机访问分别调用了下面两个方法
//private static <T> int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key){...}//私有方法，在binarySearch中进行了调用
//private static <T> int iteratorBinarySearch(List<? extends Comparable<? super T>> list, T key){...}//私有方法，在binarySearch中进行了调用
//private static <T> T get(ListIterator<? extends T> i, int index) {}//私有方法，在iteratorBinarySearch中进行了调用
public static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c) {...}////在指定的列表中使用二分查找搜索指定的元素，列表中元素必须为升序排列，按照指定的比较器c；若列表中有多个与key相同的元素，不能保证找到哪一个
//同样调用了以下两个方法
//private static <T> int indexedBinarySearch(List<? extends T> l, T key, Comparator<? super T> c) {}
//private static <T> int iteratorBinarySearch(List<? extends T> l, T key, Comparator<? super T> c) {}
public static void reverse(List<?> list) {...}//反转指定列表中元素的顺序，线性时间内完成
public static void shuffle(List<?> list) {...}//随机打乱指定列表中元素的顺序
public static void shuffle(List<?> list, Random rnd) {...}//使用指定的随机序列打乱指定列表中元素的顺序
public static void swap(List<?> list, int i, int j) {..}//在指定的列表中交换指定位置的元素
private static void swap(Object[] arr, int i, int j) {...}//在指定的数组中交换指定位置的元素
public static <T> void fill(List<? super T> list, T obj) {...}//用指定的元素替换掉指定列表中的所有元素
public static <T> void copy(List<? super T> dest, List<? extends T> src) {..}//从指定的列表中复制元素至另一个列表，index不变，即元素顺序不改变，目标列表大小必须大于等于源列表。dest-目标列表，src-源列表
public static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll) {...}//返回集合中的最小元素，根据元素的自然排序
public static <T> T min(Collection<? extends T> coll, Comparator<? super T> comp) {...}//返回集合中的最小元素，根据指定的比较器
public static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll) {...}//返回集合中的最大元素，根据元素的自然排序
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> comp) {...}//返回集合中最大元素，根据指定的比较器
public static void rotate(List<?> list, int distance) {..}//将指定列表中的元素循环移位指定距离，distance>0，循环向右移，distance<0，循环向左移，实现方式是调用了下面两个私有方法
//private static <T> void rotate1(List<T> list, int distance) {}
//private static void rotate2(List<?> list, int distance) {}
public static <T> boolean replaceAll(List<T> list, T oldVal, T newVal) {...}//用新元素（newVal）置换掉指定列表中的所有指定元素(oldVal)
public static int indexOfSubList(List<?> source, List<?> target) {..}//返回targer列表在source列表中第一次出现的位置，即满足条件  source.subList(i, i+target.size()).equals(target)  的最小i值
//没有出现或、target的大小大于source0时返回-1
public static int lastIndexOfSubList(List<?> source, List<?> target) {...}//返回targer列表在source列表中最后一次出现的位置，即满足条件  source.subList(i, i+target.size()).equals(target)  的最大i值
//没有出现或、target的大小大于source时返回-1
public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c) {...}//返回一个c的不可修改的集合，返回的集合不会将equals()方法和hashCode()方法传到底层集合
public static <T> Set<T> unmodifiableSet(Set<? extends T> s){...}//返回一个不可修改的Set集合
public static <T> SortedSet<T> unmodifiableSortedSet(SortedSet<T> s){...}//返回一个不可修改的有序Set集合
public static <T> NavigableSet<T> unmodifiableNavigableSet(NavigableSet<T> s) {...}//返回一个不可修改的NavigableSet
public static <T> List<T> unmodifiableList(List<? extends T> list) {...}//返回一个不可修改的List
public static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m){...}//返回一个不可修改的Map
public static <K,V> SortedMap<K,V> unmodifiableSortedMap(SortedMap<K, ? extends V> m) {}//返回一个不可修改的有序Map
public static <K,V> NavigableMap<K,V> unmodifiableNavigableMap(NavigableMap<K, ? extends V> m)//返回一个不可修改的NavigableMap
public static <T> Collection<T> synchronizedCollection(Collection<T> c){}//返回一个同步的（即线程安全的）Collection
public static <T> Set<T> synchronizedSet(Set<T> s){}//返回一个同步的Set
public static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s){}//返回一个同步的SortedSet
public static <T> NavigableSet<T> synchronizedNavigableSet(NavigableSet<T> s)//返回一个同步的NavigableSet
public static <T> List<T> synchronizedList(List<T> list) {}//返回一个同步List
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m){}//返回一个同步的Map
public static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m) {}//返回一个同步的SortedMap
public static <K,V> NavigableMap<K,V> synchronizedNavigableMap(NavigableMap<K,V> m) {}//返回一个同步的NavigableMap
public static <E> Collection<E> checkedCollection(Collection<E> c, Class<E> type) {}//返回指定Collection的动态类型安全视图
public static <E> Queue<E> checkedQueue(Queue<E> queue, Class<E> type){}//返回指定queue的动态类型安全视图
public static <E> Set<E> checkedSet(Set<E> s, Class<E> type) {}//返回指定set的动态类型安全视图
public static <E> SortedSet<E> checkedSortedSet(SortedSet<E> s, Class<E> type) {}//返回指定sortedSet的动态类型安全视图
public static <E> NavigableSet<E> checkedNavigableSet(NavigableSet<E> s, Class<E> type) {}//返回指定NavigableSet的动态类型安全视图
public static <E> List<E> checkedList(List<E> list, Class<E> type) {}//返回指定List的动态类型安全视图
public static <K, V> Map<K, V> checkedMap(Map<K, V> m, Class<K> keyType, Class<V> valueType){}////返回指定Map的动态类型安全视图
public static <K,V> SortedMap<K,V> checkedSortedMap(SortedMap<K, V> m, Class<K> keyType,Class<V> valueType){}//返回指定sortedMap的动态类型安全视图
public static <K,V> NavigableMap<K,V> checkedNavigableMap(NavigableMap<K, V> m,Class<K> keyType,Class<V> valueType){}//返回指定NavigableMap的动态类型安全视图
public static <T> Iterator<T> emptyIterator(){}//返回一个空的Iterator，即hasNext()返回一直为false，next()返回抛出一个NoSuchElementException，remove()返回抛出一个IllegalStateException
public static <T> ListIterator<T> emptyListIterator(){}//返回一个空的ListIterator
public static <T> Enumeration<T> emptyEnumeration(){}//返回一个空的枚举 enumeration
public static final <T> Set<T> emptySet(){}//返回一个空的不可变的的Set
public static <E> SortedSet<E> emptySortedSet() {}//返回一个空的不可变的的sortedSet
public static <E> NavigableSet<E> emptyNavigableSet() {}//返回一个空的不可变的的NavigableSet
public static final <T> List<T> emptyList(){}//返回一个空的不可变的List
public static final <K,V> Map<K,V> emptyMap(){}//返回一个空的不可变的Map
public static final <K,V> SortedMap<K,V> emptySortedMap() {}//返回一个空的不可变的sortedMap
public static final <K,V> NavigableMap<K,V> emptyNavigableMap() {}//返回一个空的不可变的NavigableMap
public static <T> Set<T> singleton(T o){}//返回一个只包含一个指定元素的不可变的Set
public static <T> List<T> singletonList(T o){}//返回一个只包含一个指定元素的不可变的List
public static <K,V> Map<K,V> singletonMap(K key, V value){}//返回一个只包含指定键、值的不可变的Map
public static <T> List<T> nCopies(int n, T o) {}//返回一个只包含n个重复指定元素的List
public static <T> Comparator<T> reverseOrder(){}//返回一个比较器，该比较器的比较规则与collection元素的自然比较规则相反
public static <T> Comparator<T> reverseOrder(Comparator<T> cmp)////返回一个比较器，该比较器的比较规则与指定比较器的比较规则相反
public static <T> Enumeration<T> enumeration(final Collection<T> c) {}//返回一个指定collection集合的枚举
public static <T> ArrayList<T> list(Enumeration<T> e) {}//Returns an array list containing the elements returned by the specified enumeration in the order they are returned by the enumeration. 
public static int frequency(Collection<?> c, Object o) {}//Returns the number of elements in the specified collection equal to the specified object.
public static boolean disjoint(Collection<?> c1, Collection<?> c2) {}//Returns {@code true} if the two specified collections have no elements in common.
public static <T> boolean addAll(Collection<? super T> c, T... elements) {}//Adds all of the specified elements to the specified collection.
public static <E> Set<E> newSetFromMap(Map<E, Boolean> map){}//Returns a set backed by the specified map.  The resulting set displays the same ordering, concurrency, and performance characteristics as the backing map.
public static <T> Queue<T> asLifoQueue(Deque<T> deque){}//Returns a view of a {@link Deque} as a Last-in-first-out (Lifo)
```

没漏的话应该是Collections工具类的所有public方法了，不行了，后面的翻译不下去了，直接复制英文注释了，还有Arrays工具类，有时间再看吧，看得脑壳儿疼