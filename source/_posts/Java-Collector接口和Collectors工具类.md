---
title: Java Collector接口和Collectors工具类
date: 2020-03-05 15:56:47
categories: java
tags:
---

参考自[原文](https://www.jianshu.com/p/7eaa0969b424)，感谢原作者

![](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020感谢.png)

### 概述

Collector是专门用于Stream流collect操作的参数的，而Collectors对Collector接口进行了实现

```java
<R, A> R collect(Collector<? super T, A, R> collector);
```

### Collector接口

#### 接口定义

```java
public interface Collector<T, A, R> {
    //返回一个新的，可变的结果容器
	Supplier<A> supplier();
    //返回一个消费函数，将元素值折叠到结果容器中
	BiConsumer<A, T> accumulator();
    //用于成对的合并并行执行的线程的执行结果，将其最终合并成一个结果A
	BinaryOperator<A> combiner();
    //用于将之前整合完成的结果R转换成A
	Function<A, R> finisher();
    //characteristics表示当前Collector的特征值，这是个不可变Set
	Set<Characteristics> characteristics();
    //下面两个静态方法用于上传Collector实例
    //四参方法，用于生成一个Collector，T代表流中的一个一个元素，R代表最终的结果
	public static<T, R> Collector<T, R, R> of(Supplier<R> supplier,BiConsumer<R, T> accumulator,BinaryOperator<R> combiner,Characteristics... characteristics) {
   
   //方法内部源码省略
        ...
            ...
     }
    //五参方法，用于生成一个Collector，T代表流中的一个一个元素，A代表中间结果，R代表最终结果，finisher用于将A转换为R
    //Characteristics：这个特征值是一个枚举，拥有三个值：CONCURRENT（多线程并行），UNORDERED（无序），IDENTITY_FINISH（无需转换结果）。其中四参of方法中没有finisher参数，所以必有IDENTITY_FINISH特征值。
     public static<T, A, R> Collector<T, A, R> of(Supplier<A> supplier,BiConsumer<A, T> accumulator,BinaryOperator<A> combiner,Function<A, R> finisher,Characteristics... characteristics) {
   //方法内部源码省略
         ...
         ...
    }
    //枚举值
    enum Characteristics {
      CONCURRENT,//多线程运行
      UNORDERED,//无序
      IDENTITY_FINISH//无需转换结果
}
```

### Collectors

Collecors是JDK实现的一个工具类，类中对于Collector接口进行了实现，并提供了多种Collector，可以直接使用

实现的Collector有：

#### toCollection

将流中元素全部放置到一个集合中返回，这里使用Collection接口，泛指多种集合

源码

```java
//接受一个Supplier用于生成集合实例
public static <T, C extends Collection<T>>
    Collector<T, ?, C> toCollection(Supplier<C> collectionFactory) {
        return new CollectorImpl<>(collectionFactory, Collection<T>::add,
                                   (r1, r2) -> { r1.addAll(r2); return r1; },
                                   CH_ID);
    }
```

使用示例

```java
public class CollectorsTest {
    public static void toCollectionTest(List<String> list) {
        List<String> ll = list.stream().collect(Collectors.toCollection(LinkedList::new));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        toCollectionTest(list);
    }
}
```

#### toList

将流中的元素至于一个列表中，默认为ArrayList列表

源码

```java
public static <T>
Collector<T, ?, List<T>> toList() {
    return new CollectorImpl<>((Supplier<List<T>>) ArrayList::new, List::add,
                               (left, right) -> { left.addAll(right); return left; },
                               CH_ID);
}
```

使用实例

```java
public class CollectorsTest {
    public static void toListTest(List<String> list) {
        List<String> ll = list.stream().collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        toListTest(list);
    }
}
```

#### toSet

将流中元素置于到一个无需Set中，默认为HashSet

源码

```java
public static <T>
Collector<T, ?, Set<T>> toSet() {
    return new CollectorImpl<>((Supplier<Set<T>>) HashSet::new, Set::add,
                               (left, right) -> { left.addAll(right); return left; },
                               CH_UNORDERED_ID);
}
```

使用实例

```java
public class CollectorsTest {
    public static void toSetTest(List<String> list) {
        Set<String> ss = list.stream().collect(Collectors.toSet());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        toSetTest(list);
    }
}
```

#### joining

目的是将流中的元素全部以字符序列的方式连接在一起，可以指定连接符，和结果的前后缀

有三种重载方法

源码

```java
//
public static Collector<CharSequence, ?, String> joining() {
    return new CollectorImpl<CharSequence, StringBuilder, String>(
            StringBuilder::new, StringBuilder::append,
            (r1, r2) -> { r1.append(r2); return r1; },
            StringBuilder::toString, CH_NOID);
}
//指定连接符
public static Collector<CharSequence, ?, String> joining(CharSequence delimiter) {
        return joining(delimiter, "", "");
}
//指定连接符和结果前后缀
//StringJoiner：一个字符串连接器，可以定义连接符和前后缀
public static Collector<CharSequence, ?, String> joining(CharSequence delimiter,
                                                             CharSequence prefix,
                                                             CharSequence suffix) {
        return new CollectorImpl<>(
                () -> new StringJoiner(delimiter, prefix, suffix),
                StringJoiner::add, StringJoiner::merge,
                StringJoiner::toString, CH_NOID);
}
```

使用实例

```java
public class CollectorsTest {
    public static void joiningTest(List<String> list){
        // 无参方法
        String s = list.stream().collect(Collectors.joining());
        System.out.println(s);
        // 指定连接符
        String ss = list.stream().collect(Collectors.joining("-"));
        System.out.println(ss);
        // 指定连接符和前后缀
        String sss = list.stream().collect(Collectors.joining("-","S","E"));
        System.out.println(sss);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        joiningTest(list);
    }
}
```

执行结果

```text
1234567891101212121121asdaa3e3e3e2321eew
123-456-789-1101-212121121-asdaa-3e3e3e-2321eew
S123-456-789-1101-212121121-asdaa-3e3e3e-2321eewE
```

#### mapping

这个映射是首先对流中的每个元素进行映射，即类型转换，然后再将新元素以给定的Collector进行归纳

源码

```java
public static <T, U, A, R>
Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper,
                           Collector<? super U, A, R> downstream) {
    BiConsumer<A, ? super U> downstreamAccumulator = downstream.accumulator();
    return new CollectorImpl<>(downstream.supplier(),
                               (r, t) -> downstreamAccumulator.accept(r, mapper.apply(t)),
                               downstream.combiner(), downstream.finisher(),
                               downstream.characteristics());
}
```

使用实例

```java
//截取字符串列表的前5个元素，将其分别转换为Integer类型，然后放到一个List中返回
public class CollectorsTest {
    public static void mapingTest(List<String> list){
        List<Integer> ll = list.stream().limit(5).collect(Collectors.mapping(Integer::valueOf,Collectors.toList()));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        mapingTest(list);
    }
}
```

#### collectingAndThen

该方法是在归纳动作结束之后，对归纳的结果进行再处理

源码

```java
public static<T,A,R,RR> Collector<T,A,RR> collectingAndThen(Collector<T,A,R> downstream,
                                                                Function<R,RR> finisher) {
        Set<Collector.Characteristics> characteristics = downstream.characteristics();
        if (characteristics.contains(Collector.Characteristics.IDENTITY_FINISH)) {
            if (characteristics.size() == 1)
                characteristics = Collectors.CH_NOID;
            else {
                characteristics = EnumSet.copyOf(characteristics);
                characteristics.remove(Collector.Characteristics.IDENTITY_FINISH);
                characteristics = Collections.unmodifiableSet(characteristics);
            }
        }
        return new CollectorImpl<>(downstream.supplier(),
                                   downstream.accumulator(),
                                   downstream.combiner(),
                                   downstream.finisher().andThen(finisher),
                                   characteristics);
    }
```

使用实例

```java
public class CollectorsTest {
    public static void collectingAndThenTest(List<String> list){
        int length = list.stream().collect(Collectors.collectingAndThen(Collectors.toList(),e -> e.size()));
        System.out.println(length);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        collectingAndThenTest(list);
    }
}
```

执行结果

```text
8
```

#### counting

用于计数

源码

```java
public static <T> Collector<T, ?, Long>
counting() {
    return reducing(0L, e -> 1L, Long::sum);
}
```

使用实例

```java
public class CollectorsTest {
    public static void countingTest(List<String> list){
        long size = list.stream().collect(Collectors.counting());
        System.out.println(size);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        countingTest(list);
    }
}
```

执行结果

```text
8
```

#### minBy/maxBy

生成一个用于获取最小/最大值的Optional结果的Collector

源码

```java
public static <T> Collector<T, ?, Optional<T>>
    minBy(Comparator<? super T> comparator) {
        return reducing(BinaryOperator.minBy(comparator));
    }
```

```java
public static <T> Collector<T, ?, Optional<T>>
maxBy(Comparator<? super T> comparator) {
    return reducing(BinaryOperator.maxBy(comparator));
}
```

使用实例

```java
public class CollectorsTest {
    public static void maxByAndMinByTest(List<String> list){
        System.out.println(list.stream().collect(Collectors.maxBy((a,b) -> a.length()-b.length())));
        System.out.println(list.stream().collect(Collectors.minBy((a,b) -> a.length()-b.length())));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        maxByAndMinByTest(list);
    }
}
```

执行结果

```text
Optional[212121121]
Optional[123]
```

#### summingInt/summingLong/summingDouble

生成一个用于元素求和的Collector，首先通过给定的函数参数（mapper）对元素类型进行转换，然后再求和

参数为一个Function，针对不同的类型有不同的Function，参数的作用就是将元素转换为指定的类型，最后结果与转换后类型一致

源码

```java
public static <T> Collector<T, ?, Integer>
summingInt(ToIntFunction<? super T> mapper) {
    return new CollectorImpl<>(
            () -> new int[1],
            (a, t) -> { a[0] += mapper.applyAsInt(t); },
            (a, b) -> { a[0] += b[0]; return a; },
            a -> a[0], CH_NOID);
}
```

```java
public static <T> Collector<T, ?, Long>
summingLong(ToLongFunction<? super T> mapper) {
    return new CollectorImpl<>(
            () -> new long[1],
            (a, t) -> { a[0] += mapper.applyAsLong(t); },
            (a, b) -> { a[0] += b[0]; return a; },
            a -> a[0], CH_NOID);
}
```

```java
public static <T> Collector<T, ?, Double>
summingDouble(ToDoubleFunction<? super T> mapper) {
   
    return new CollectorImpl<>(
            () -> new double[3],
            (a, t) -> { sumWithCompensation(a, mapper.applyAsDouble(t));
                        a[2] += mapper.applyAsDouble(t);},
            (a, b) -> { sumWithCompensation(a, b[0]);
                        a[2] += b[2];
                        return sumWithCompensation(a, b[1]); },
            a -> computeFinalSum(a),
            CH_NOID);
}
```

使用实例

```java
public class CollectorsTest {
    public static void summingTest(List<String> list){
        int i = list.stream().limit(3).collect(Collectors.summingInt(Integer::valueOf));
        long l = list.stream().limit(3).collect(Collectors.summingLong(Long::valueOf));
        double d = list.stream().limit(3).collect(Collectors.summingDouble(Double::valueOf));
        System.out.println(i +"\n" +l + "\n" + d);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        summingTest(list);
    }
}
```

执行结果为

```text
1368
1368
1368.0
```

#### averagingInt/averagingLong/averagingDouble

生成一个用于求元素平均值的Collector，首先通过参数将元素转换为指定的类型

参数也为一个Function，作用就是将元素转换为指定的类型，因为求平均值涉及到除法操作，所以结果一律为Double型

源码

```java
public static <T> Collector<T, ?, Double>
averagingInt(ToIntFunction<? super T> mapper) {
    return new CollectorImpl<>(
            () -> new long[2],
            (a, t) -> { a[0] += mapper.applyAsInt(t); a[1]++; },
            (a, b) -> { a[0] += b[0]; a[1] += b[1]; return a; },
            a -> (a[1] == 0) ? 0.0d : (double) a[0] / a[1], CH_NOID);
}
```

```java
public static <T> Collector<T, ?, Double>
averagingLong(ToLongFunction<? super T> mapper) {
    return new CollectorImpl<>(
            () -> new long[2],
            (a, t) -> { a[0] += mapper.applyAsLong(t); a[1]++; },
            (a, b) -> { a[0] += b[0]; a[1] += b[1]; return a; },
            a -> (a[1] == 0) ? 0.0d : (double) a[0] / a[1], CH_NOID);
}
```

```java
public static <T> Collector<T, ?, Double>
averagingDouble(ToDoubleFunction<? super T> mapper) {
  
    return new CollectorImpl<>(
            () -> new double[4],
            (a, t) -> { sumWithCompensation(a, mapper.applyAsDouble(t)); a[2]++; a[3]+= mapper.applyAsDouble(t);},
            (a, b) -> { sumWithCompensation(a, b[0]); sumWithCompensation(a, b[1]); a[2] += b[2]; a[3] += b[3]; return a; },
            a -> (a[2] == 0) ? 0.0d : (computeFinalSum(a) / a[2]),
            CH_NOID);
}
```

使用实例

```java
public class CollectorsTest {
    public static void averagingTest(List<String> list){
        double i = list.stream().limit(3).collect(Collectors.averagingInt(Integer::valueOf));
        double l = list.stream().limit(3).collect(Collectors.averagingLong(Long::valueOf));
        double d = list.stream().limit(3).collect(Collectors.averagingDouble(Double::valueOf));
        System.out.println(i +"\n" +l + "\n" + d);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        averagingTest(list);
    }
}
```

执行结果为

```text
456.0
456.0
456.0
```

#### reducing

reducing方法有三个重载方法，其实是和Stream里的三个reduce方法对应的，二者是可以替换使用的，作用完全一致，也是对流中的元素做统计归纳作用

源码

```java
public static <T> Collector<T, ?, T>
reducing(T identity, BinaryOperator<T> op) {
     //内部源码省略
    ...
    ...
}
```

```java
public static <T> Collector<T, ?, Optional<T>>
reducing(BinaryOperator<T> op) {
     //内部源码省略
    ...
    ...
}
```

```java
public static <T> Collector<T, ?, Optional<T>>
reducing(BinaryOperator<T> op) {
    //内部源码省略
    ...
    ...
}
```

使用实例

```java
public class CollectorsTest {
    public static void reducingTest(List<String> list){
        System.out.println(list.stream().limit(4).map(String::length).collect(Collectors.reducing(Integer::sum)));
        System.out.println(list.stream().limit(3).map(String::length).collect(Collectors.reducing(0, Integer::sum)));
        System.out.println(list.stream().limit(4).collect(Collectors.reducing(0,String::length,Integer::sum)));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        reducingTest(list);
    }
}
```

执行结果为

```text
Optional[13]
9
13
```

#### groupingBy

用于生成一个拥有分组功能的Collector，有三个重载方法

源码

```java
// 只需一个分组参数classifier，内部自动将结果保存到一个map中，每个map的键为?类型（即classifier的结果类型），值为一个list，这个list中保存在属于这个组的元素。
public static <T, K> Collector<T, ?, Map<K, List<T>>>
groupingBy(Function<? super T, ? extends K> classifier) {
    return groupingBy(classifier, toList());
}
```

```java
// 在上面方法的基础上增加了对流中元素的处理方式的Collector，比如上面的默认的处理方法就是Collectors.toList()
public static <T, K, A, D>
Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                      Collector<? super T, A, D> downstream) {
    return groupingBy(classifier, HashMap::new, downstream);
}
```

```java
// 在第二个方法的基础上再添加了结果Map的生成方法。
public static <T, K, D, A, M extends Map<K, D>>
Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,Supplier<M> mapFactory,Collector<? super T, A, D> downstream) {
             //内部源码省略
    ...
    ...
                              }
```

使用实例

```java
public class CollectorsTest {
    public static void groupingByTest(List<String> list){
        Map<Integer,List<String>> s = list.stream().collect(Collectors.groupingBy(String::length));
        Map<Integer,List<String>> ss = list.stream().collect(Collectors.groupingBy(String::length, Collectors.toList()));
        Map<Integer,Set<String>> sss = list.stream().collect(Collectors.groupingBy(String::length,HashMap::new,Collectors.toSet()));
        System.out.println(s.toString() + "\n" + ss.toString() + "\n" + sss.toString());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        groupingByTest(list);
    }
}
```

执行结果

```text
{3=[123, 456, 789], 4=[1101], 5=[asdaa], 6=[3e3e3e], 7=[2321eew], 9=[212121121]}
{3=[123, 456, 789], 4=[1101], 5=[asdaa], 6=[3e3e3e], 7=[2321eew], 9=[212121121]}
{3=[123, 456, 789], 4=[1101], 5=[asdaa], 6=[3e3e3e], 7=[2321eew], 9=[212121121]}
```

#### groupingByConcurrent

也有3个重载方法，功能与groupingBy基本一致，但返回的Collector并行执行

#### partioningBy

将流中的元素按照指定的校验规则的结果分为两个部分，放到一个Map中返回，map的键是Boolean类型，值为元素的列表List

有两个重载方法

```java
public static <T>
Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate) {
    return partitioningBy(predicate, toList());
}
```

```java
public static <T, D, A>
Collector<T, ?, Map<Boolean, D>> partitioningBy(Predicate<? super T> predicate,
                                                Collector<? super T, A, D> downstream) {
           //内部源码省略
    ...
    ...                 
                                                }
```

使用实例

```java
public class CollectorsTest {
    public static void partitioningByTest(List<String> list){
        Map<Boolean,List<String>> map = list.stream().collect(Collectors.partitioningBy(e -> e.length()>5));
        Map<Boolean,Set<String>> map2 = list.stream().collect(Collectors.partitioningBy(e -> e.length()>6,Collectors.toSet()));
        System.out.println(map.toString() + "\n" + map2.toString());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        partitioningByTest(list);
    }
}
```

执行结果

```text
{false=[123, 456, 789, 1101, asdaa], true=[212121121, 3e3e3e, 2321eew]}
{false=[123, 456, 1101, 789, 3e3e3e, asdaa], true=[212121121, 2321eew]}
```

#### toMap

该方法时根据指定的键生成器和值生成器生成的键和值保存到一个map中返回，键和值的生成都依赖于元素，可以指定出现重复键时的处理方式和保存结果的Map

有三个重载方法

源码

```java
// 指定键和值的生成方式keyMapper和valueMapper
public static <T, K, U>
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper) {
    return toMap(keyMapper, valueMapper, throwingMerger(), HashMap::new);
}
```

```java
// 在上面方法的基础上增加了对键发生重复时处理方式的mergeFunction，比如上面的默认的处理方法就是抛出异常
public static <T, K, U>
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper,
                                BinaryOperator<U> mergeFunction) {
    return toMap(keyMapper, valueMapper, mergeFunction, HashMap::new);
}
```

```java
 // 在第二个方法的基础上再添加了结果Map的生成方法。
public static <T, K, U, M extends Map<K, U>>
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                            Function<? super T, ? extends U> valueMapper,
                            BinaryOperator<U> mergeFunction,
                            Supplier<M> mapSupplier) {
                             //内部源码省略
    ...
    ...
                            }
```

使用实例

```java
public class CollectorsTest {
    public static void toMapTest(List<String> list){
        Map<String,String> map = list.stream().limit(3).collect(Collectors.toMap(e -> e.substring(0,1),e -> e));
        Map<String,String> map1 = list.stream().collect(Collectors.toMap(e -> e.substring(0,1),e->e,(a,b)-> b));
        Map<String,String> map2 = list.stream().collect(Collectors.toMap(e -> e.substring(0,1),e->e,(a,b)-> b,HashMap::new));
        System.out.println(map.toString() + "\n" + map1.toString() + "\n" + map2.toString());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        toMapTest(list);
    }
}
```

执行结果

```text
{1=123, 4=456, 7=789}
{a=asdaa, 1=1101, 2=2321eew, 3=3e3e3e, 4=456, 7=789}
{a=asdaa, 1=1101, 2=2321eew, 3=3e3e3e, 4=456, 7=789}
```

#### toConcurrentMap

有三种重载方法，与toMap基本一致，只是它最后使用的map是并发Map:ConcurrentHashMap

#### summarizingInt/summarizingLong/summarizingDouble

这三个方法适用于汇总的，返回值分别是IntSummaryStatistics，LongSummaryStatistics，DoubleSummaryStatistics。

在这些返回值中包含有流中元素的指定结果的数量、和、最大值、最小值、平均值。所有仅仅针对数值结果

源码

```java
public static <T>
Collector<T, ?, IntSummaryStatistics> summarizingInt(ToIntFunction<? super T> mapper) {
    return new CollectorImpl<T, IntSummaryStatistics, IntSummaryStatistics>(
            IntSummaryStatistics::new,
            (r, t) -> r.accept(mapper.applyAsInt(t)),
            (l, r) -> { l.combine(r); return l; }, CH_ID);
}
```

```java
public static <T>
Collector<T, ?, LongSummaryStatistics> summarizingLong(ToLongFunction<? super T> mapper) {
    return new CollectorImpl<T, LongSummaryStatistics, LongSummaryStatistics>(
            LongSummaryStatistics::new,
            (r, t) -> r.accept(mapper.applyAsLong(t)),
            (l, r) -> { l.combine(r); return l; }, CH_ID);
}
```

```java
public static <T>
Collector<T, ?, DoubleSummaryStatistics> summarizingDouble(ToDoubleFunction<? super T> mapper) {
    return new CollectorImpl<T, DoubleSummaryStatistics, DoubleSummaryStatistics>(
            DoubleSummaryStatistics::new,
            (r, t) -> r.accept(mapper.applyAsDouble(t)),
            (l, r) -> { l.combine(r); return l; }, CH_ID);
}
```

使用实例

```java
public class CollectorsTest {
    public static void summarizingTest(List<String> list){
        IntSummaryStatistics intSummary = list.stream().collect(Collectors.summarizingInt(String::length));
        LongSummaryStatistics longSummary = list.stream().limit(4).collect(Collectors.summarizingLong(Long::valueOf));
        DoubleSummaryStatistics doubleSummary = list.stream().limit(3).collect(Collectors.summarizingDouble(Double::valueOf));
        System.out.println(intSummary.toString() + "\n" + longSummary.toString() + "\n" + doubleSummary.toString());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        summarizingTest(list);
    }
}
```

执行结果

```text
IntSummaryStatistics{count=8, sum=40, min=3, average=5.000000, max=9}
LongSummaryStatistics{count=4, sum=2469, min=123, average=617.250000, max=1101}
DoubleSummaryStatistics{count=3, sum=1368.000000, min=123.000000, average=456.000000, max=789.000000}
```

### 总结

Colletcors工具类就是提供多种Collector，作为Stream流操作处理中终结操作collect的参数

有些功能与Stream流中的操作重合，为了简化代码，优先使用Stream中的方法

以上实例代码均来自[此文](https://www.jianshu.com/p/7eaa0969b424)，再次感谢大佬！

![](https://cdn.jsdelivr.net/gh/yanzixian/figureBed/img2020感谢2.png)

