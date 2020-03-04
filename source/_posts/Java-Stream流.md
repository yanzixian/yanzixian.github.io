---
title: Java Stream流
date: 2020-03-02 15:41:50
categories: java
tags:

---

## Java 8 Stream

### 概述

> Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。
>
> Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。
>
> Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。
>
> 这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。
>
> 元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

```java
+--------------------+       +------+   +------+   +---+   +-------+
| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|
+--------------------+       +------+   +------+   +---+   +-------+
```

Stream接口定义源码：

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
	...
	...
	}
```



### [什么是Stream?](https://www.runoob.com/java/java8-streams.html)

Stream（流）是一个来自数据源的元素队列并支持聚合操作

- 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

- **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
- **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

### 生成流

- Stream接口提供了多种静态方法用于创建流

  源码

  ```java
  //创建流时需要指定类型
  public static<T> Builder<T> builder() {
          return new Streams.StreamBuilderImpl<>();
      }
  
  //创建一个空流，避免为没有元素的流返回Null导致异常
  public static<T> Stream<T> empty() {
          return StreamSupport.stream(Spliterators.<T>emptySpliterator(), false);
      }
  //为单个元素创建流
  public static<T> Stream<T> of(T t) {
          return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
      }
  //为多个元素创建流
  public static<T> Stream<T> of(T... values) {
          return Arrays.stream(values);
      }
  //使用iterate方法
  public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
  	...
  	...
  	...
  }
  
  //需要接受一个Supplier作为元素生成，还需指定流的大小
  public static<T> Stream<T> generate(Supplier<T> s) {
          Objects.requireNonNull(s);
          return StreamSupport.stream(new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef<>(Long.MAX_VALUE, s), false);
  }
  //接受两个Stream参数，返回两个流连接组成的新流
      public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b) {
      ...
      ...
      ...
  }
  ```

  示例

  ```java
  Stream<String> streamBuilder = Stream.<String>builder().add("a").add("b").add("c").build();
  
  Stream<String> streamEmpty = Stream.empty();
  
  Stream<String> streamOfArray = Stream.of("a", "b", "c");
  
  Stream<Integer> streamIterated = Stream.iterate(40, n -> n + 2).limit(20);
  
  Stream<String> streamGenerated =Stream.generate(() -> "element").limit(10);
  ```

  

- Java 8 中，Collection集合接口有两个方法来生成流：
  - stream()   为集合创建**串行**流
  - parallelStream()  为集合创建**并行**流

- Arrays工具类提供了多个静态方法由数组创建流

  ```java
  public static <T> Stream<T> stream(T[] array) {
          return stream(array, 0, array.length);
      }
  //其余省略
  ...
  ...
  ...
  ```

  示例如下

[参考](https://www.jianshu.com/p/3dc56886c2eb)

#### 基于数组创建流

```java
public class StreamTest {
    public static void createStream() {
        // 通过数组生成流
        int[] ints = {1,2,3,4,5,6};
        IntStream s1 = Arrays.stream(ints);
        Stream s2 = Stream.of("111","222","333");
        String[] ss = {"123","321","456","654"};
        Stream<String> s3 = Arrays.stream(ss);
    }
}
```

#### 通过构建器生成流

```java
public class StreamTest {
    public static void createStream() {
        // 通过构建器生成流
        Stream<Object> s4 = Stream.builder().add("123").add("321").add("444").add("@21").build();
    }
}
```

#### 基于集合生成流

```java
public class StreamTest {
    public static void createStream() {
        // 通过集合生成流
        List<String> lists = Arrays.asList("123","321","1212","32321");
        Stream<String> s5 = lists.stream();
        Stream<String> s6 = lists.parallelStream();// 并行流
    }
}
```

#### 创建空流

```java
public class StreamTest {
    public static void createStream() {
        // 创建空流
        Stream<String> s7  = Stream.empty();
    }
}
```

#### 基于函数创建流

```java
public class StreamTest {
    public static void createStream() {
        // 创建无限流
        Stream.generate(()->"number"+new Random().nextInt()).limit(100).forEach(System.out::println);
        Stream.iterate(0,n -> n+2).limit(10).forEach(System.out::println);
    }
}
```

### 流中间操作

中间操作的返回值仍然是流

#### 列表汇总

| 方法(省略返回类型和参数) | 说明                                                 |
| ------------------------ | ---------------------------------------------------- |
| filter()                 | 过滤特定元素后的流返回                               |
| map()                    | 将给定mapper函数作用于每个元素，组成新流返回         |
| mapToInt()               | 返回将给定mapper函数作用于每个元素组成的新的Int流    |
| mapToLong()              | 返回将给定mapper函数作用于每个元素组成的新的Long流   |
| mapToDouble()            | 返回将给定mapper函数作用于每个元素组成的新的Double流 |
| flatMap()                | 将给定mapper函数作用于每个元素，组成新流返回         |
| flatMapToInt()           | 将给定mapper函数作用于每个元素，组成新的Int流返回    |
| flatMapToLong()          | 将给定mapper函数作用于每个元素，组成新的Long流返回   |
| flatMapToDouble()        | 将给定mapper函数作用于每个元素，组成新的Double流返回 |
| distinct()               | 去掉重复元素，返回新流                               |
| sorted()                 | 将元素排序后返回新流                                 |
| sorted()（指定比较器）   | 将元素按指定比较器排序后返回新流                     |
| peek()                   | 针对流中的每个元素执行操作                           |
| limit()                  | 返回指定数量的元素组成的新流                         |
| skip()                   | 返回第n个元素之后的元素组成的新流                    |

以下代码示例均来自[此文](https://www.jianshu.com/p/3dc56886c2eb)，感谢原文大佬

#### filter

filter方法实现对流中元素的过滤，满足条件的元素组成新流返回

接口方法定义源码

该方法接受一个Predicate参数，作为过滤条件

```java
  Stream<T> filter(Predicate<? super T> predicate);
```

使用

```java
public class StreamTest {
    public static void filterTest(List<String> list){
        list.stream()
                .filter(e -> e.length() > 4 && e.length()<7)// 过滤掉长度小于等于4,大于等于7的元素
                .peek(System.out::println)// 查阅中间流结果
                .collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        filterTest(list);
    }
}
```

  执行结果为：

```
asdaa
3e3e3e
```

#### map

接口方法定义源码

此方法接受一个Function参数，对每一个元素进行处理

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

使用

```java
public class StreamTest {
    public static void mapTest(List<String> list){
        list.stream()
                .map(e -> "@" + e)// 为每个元素执行操作：添加前缀
                .peek(System.out::println)// 查阅中间流结果
                .collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        mapTest(list);
    }
}
```

mapToInt、mapToLong、mapToDouble方法是map方法的扩展，源码如下：

```java
IntStream mapToInt(ToIntFunction<? super T> mapper);
LongStream mapToLong(ToLongFunction<? super T> mapper);
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);
```

其参数分别为ToIntFunction、ToLongFunction、ToDoubleFunction，分别接口一个参数，返回指定类型的值，分别为int、long、double，那么定义方法的时候就要注意返回值的类型了，必须一致，最后组成的新流就是一个int或long或double元素流（IntStream、LongStream、DoubleStream）

mapToInt使用示例（其他类似）：

```java
public class StreamTest {
    public static void mapToIntTest(List<String> list){
        list.stream()
                .mapToInt(e -> e.length())// 以元素的长度为新流
                .peek(System.out::println)// 查询中间结果
                .toArray();
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        mapToIntTest(list);
    }
}
```

执行结果为：

```java
3
3
3
4
5
6
7
9
```

#### flatMap

flatMap于map类似，都是对流中的每一个元素进行处理并返回一个新流，但不同的是flatMap会对流中元素进行扩展，即当流中元素包含子元素时，会将流中元素中的子元素提取处理，将子元素组成新流返回

源码

```java
 <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

使用

```java
public class StreamTest {
    public static void flatMap(List<String> list){
        list.stream()
                .filter(e -> e.length()>5 && e.length()<7)
                .peek(System.out::println)
                .map(e -> e.split(""))// 将每个字符串元素分解为字符数组
                .flatMap(Arrays::stream)//将每个字符数组并转化为流
                .peek(System.out::println)
                .collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        flatMap(list);
    }
}
```

执行结果为：

```
3e3e3e
3
e
3
e
3
e
```

flatMapToInt、flatMapToLong、flatMapToDouble与之前类似

#### distinct

此方法用于去重

源码

```java
Stream<T> distinct();
```

使用

```java
public class StreamTest {
    public static void distinctTest(){
        int[] int1 = {1,2,3,4};
        int[] int2 = {5,3,7,1};
        List<int[]> ints = Arrays.asList(int1,int2);
        ints.stream()
                .flatMapToInt(Arrays::stream)
                .distinct()
                .peek(System.out::println)
                .toArray();
    }
    public static void main(String[] args) {
        distinctTest();
    }
}
```

执行结果为

```
1
2
3
4
5
7
```

#### sorted

此方法用于对流中元素进行排序，无参数时使用自然排序，有比较器参数时使用指定的比较器进行排序

源码

```java
 Stream<T> sorted();
 Stream<T> sorted(Comparator<? super T> comparator);
```

使用

```java
public class StreamTest {
    public static void sortedTest(List<String> list){
        System.out.println("----自然顺序:");
        list.stream().sorted().peek(System.out::println).collect(Collectors.toList());
        System.out.println("----指定排序:");
        list.stream().sorted((a,b) -> a.length()-b.length()).peek(System.out::println).collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        sortedTest(list);
    }
}
```

执行结果为

```
----自然顺序:
1101
123
212121121
2321eew
3e3e3e
456
789
asdaa
----指定排序:
123
456
789
1101
asdaa
3e3e3e
2321eew
212121121
```

#### peek

此方法接受一个Consumer参数，该参数对于元素进行处理，但没有返回值，常用于输出

源码

```java
Stream<T> peek(Consumer<? super T> action);
```

使用

见上述示例中的peek方法使用

#### limit

用于从首个元素开始截取N个元素，组成新流返回

源码

```java
Stream<T> limit(long maxSize);
```

使用

```java
public class StreamTest {
    public static void limitTest(List<String> list){
        list.stream().limit(2).peek(System.out::println).collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        limitTest(list);
    }
}
```

执行结果为

```text
123
456
```

#### skip

此方法用于去掉前N个元素，将剩余的元素组成新流返回

源码

```java
Stream<T> skip(long n);
```

使用

```java
public class StreamTest {
    public static void skipTest(List<String> list){
        list.stream().skip(2).peek(System.out::println).collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        skipTest(list);
    }
}
```

执行结果为

```text
789
1101
asdaa
3e3e3e
2321eew
212121121
```

### 流终结操作

| 方法（省略返回和参数） | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| forEach                | 对流中的每个元素执行指定的操作                               |
| forEachOrdered         | 对流中的每个元素按序（如果有）执行指定的操作                 |
| toArray（无参）        | 返回一个包含流中所有元素的数组                               |
| toArray（有参）        | 返回一个包含流中所有元素的数组（指定类型）                   |
| reduce（2个参数）      | 以给定初始值为基础归纳流中元素，返回一个值                   |
| reduce（1个参数）      | 直接归纳流中的元素，返回一个封装有结果的Optional             |
| reduce（3个参数）      | 以给定的初始值为基础，（并行）归纳流中元素，最后将各个线程的结果再统一归纳，返回一个值 |
| collect（3个参数）     | 根据给定的各个参数归纳元素                                   |
| collect（1个参数）     | 根据给定的收集器收集元素                                     |
| min                    | 根据给定的比较器，返回流中最小元素的Optional表示             |
| max                    | 根据给定的比较器，返回流中最大元素的Optional表示             |
| count                  | 返回流中元素的个数                                           |
| anyMatch               | 校验流中是否有满足给定条件的元素                             |
| allMatch               | 校验流中的元素是否全部满足给定条件                           |
| noneMatch              | 校验流中的元素是否全不满足给点条件                           |
| findFirst              | 返回首个元素的Optional表示，如果为空流，返回空的Optional     |
| findAny                | 如果流中有元素，则返回第一个元素的Optional表示，否则返回一个空的Optional |

以下代码示例均来自[此文](https://www.jianshu.com/p/3dc56886c2eb)，再次感谢原文大佬

#### forEach和forEachOrdered

这两种方法均是遍历操作流中每个元素，做最后的操作

源码

```java
void forEach(Consumer<? super T> action);
void forEachOrdered(Consumer<? super T> action);
```

使用

```java
public class StreamTest {
    public static void forEachTest(List<String> list){
        list.stream().parallel().forEach(System.out::println);
    }
    public static void forEachOrderedTest(List<String> list){
        list.stream().parallel().forEachOrdered(System.out::println);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        forEachTest(list);
        System.out.println("----------");
        forEachOrderedTest(list);
    }
}
```

执行结果为

```text
asdaa
212121121
789
1101
2321eew
3e3e3e
456
123
----------
123
456
789
1101
212121121
asdaa
3e3e3e
2321eew
```

二者都是遍历操作，从结果是可以看出来，如果是单线程（也就是不加parallel方法的情况）那么二者结果是一致的，但是如果采用并行遍历，那么就有区别了，forEach并行遍历不保证顺序（顺序随机）,forEachOrdered却是保证顺序来进行遍历的。

#### toArray

有两个方法，一个无参，一个有参，均是返回流中元素组成的数组

源码

```java
Object[] toArray();
<A> A[] toArray(IntFunction<A[]> generator);
```

使用

```java
public class StreamTest {
    public static void toArrayTest(List<String> list){
        Object[] objs = list.stream().filter(e -> e.length()>6).toArray();
        String[] ss = list.stream().filter(e -> e.length()>6).toArray(String[]::new);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        toArrayTest(list);
    }
}
```

无参方法返回的只能是Object[]数组类型，而有参方法，可以指定结果数组类型，此乃二者区别。

使用有参方法可以直接完成类型转换，一次到位

#### reduce

此方法有三个重载方法，分别有1个，2个，3个参数，作用都是进行归纳

源码

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
    Optional<T> reduce(BinaryOperator<T> accumulator);// 编号1
    T reduce(T identity, BinaryOperator<T> accumulator);// 编号2
    <U> U reduce(U identity,BiFunction<U, ? super T, U> accumulator,BinaryOperator<U> combiner);// 编号3
}
```

首先看编号1方法，只有一个参数accumulator，这是一个累加器，方法的作用就是将这个累加器作用到流中的每一个元素，它需要两个输入参数，有一个输出参数，意思是对两个元素执行某些操作，返回一个结果，然后将这个结果与下一个元素作为参数再输入该方法，执行操作后再返回一个新结果，以此类推，直到最后一个元素执行完毕，返回的就是最终结果，因为流中的元素我们是不确定的，那么我们就无法确定reduce的结果，因为如果流为空，那么将会返回null，所以使用Optional作为返回值，妥善处理null值。

再看编号2方法，在编号1方法的基础上加了一个identity，且不再使用Optional，为什么呢，因为新加的identity其实是个初始值，后续的操作都在这个值基础上执行，那么也就是说，，如果流中没有元素的话，还有初始值作为结果返回，不会存在null的情况，也就不用Optional了。

再看编号3方法，在编号2方法的基础上又加了一个参数combiner，其实这个方法是用于处理并行流的归纳操作，最后的参数combiner用于归纳各个并行的结果，用于得出最终结果。

那么如果不使用并行流，一般使用编号2方法就足够了。

使用示例：

```java
public class StreamTest {
    public static void reduceTest(){
        List<Integer> ints = Arrays.asList(1,2,3,4,5,6,7,8,9);
        Optional<Integer> optional = ints.stream().reduce(Integer::sum);
        System.out.println(optional.get());
        System.out.println("-------------");
        Integer max = ints.stream().reduce(Integer.MIN_VALUE, Integer::max);
        System.out.println(max);
        System.out.println("-------------");
        Integer min = ints.parallelStream().reduce(Integer.MAX_VALUE, Integer::min, Integer::min);
        System.out.println(min);
    }
    public static void main(String[] args) {
        reduceTest();
    }
}
```

执行结果为：

```text
45
-------------
9
-------------
1
```

#### collect

此方法时Stream最常用的方法之一，作用就是收集归纳，将流中的数据映射为各种结果，有两个重载方法

源码

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
    <R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);// 编号1
    <R, A> R collect(Collector<? super T, A, R> collector);// 编号2
}
```

首先看看编号1方法，有三个参数：supplier用于生成一个R类型的结果容器来盛放结果，accumulator累加器用于定义盛放的方式，其中T为一个元素，R为结果容器，第三个参数combiner的作用是将并行操作的各个结果整合起来

示例

```java
public class StreamTest {
    public static void collectTest1(List<String> list){
        ArrayList<String> arrayList = list.stream().skip(4).collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
        arrayList.forEach(System.out::println);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        collectTest1(list);
    }
}
```

执行结果为：

```text
212121121
asdaa
3e3e3e
2321eew
```

上例中，第一个：ArrayList::new表示创建一个新的ArrayList集合，第二个 ArrayList::add表示将元素一个一个添加到之前的集合中，第三个ArrayList::addAll表示将多个线程的ArrayList集合一个一个的整体添加到第一个集合中，最终整合出一个最终结果并返回

编号2方法

它只需要一个Collector类型的参数，这个Collector可以称呼为收集器，我们可以随意组装一个收集器来进行元素归纳。

Collector是定义来承载一个收集器，但是JDK提供了一个Collectors工具类，在这个工具类里面预实现了N多的Collector供我们直接使用，之前的Collectors.toList()就是其用法之一

使用示例

```java
public class StreamTest {
    public static void collectTest2(List<String> list){
        Set<String> set = list.stream().skip(4).collect(Collectors.toSet());
        set.forEach(System.out::println);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        collectTest2(list);
    }
}
```

执行结果为：

```text
212121121
2321eew
3e3e3e
asdaa
```

#### max/min

通过给定的比较器，得出流中最大或最小的元素，同时使用了Optional来解决空指针异常

源码

```java
Optional<T> min(Comparator<? super T> comparator);
Optional<T> max(Comparator<? super T> comparator);
```

使用示例：

```java
public class StreamTest {
    public static void maxMinTest(List<String> list){
        System.out.println("长度最大：" + list.stream().max((a,b)-> a.length()-b.length()));
        System.out.println("长度最小：" + list.stream().min((a,b)-> a.length()-b.length()));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        maxMinTest(list);
    }
}
```

执行结果为：

```text
长度最大：Optional[212121121]
长度最小：Optional[123]
```

#### count

此方法用于计数，返回流中元素个数

源码

```java
long count();
```

使用示例：

```java
public class StreamTest {
    public static void countTest(List<String> list){
        System.out.println("元素个数为：" + list.stream().count());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        countTest(list);
    }
}
```

执行结果为：

```text
元素个数为：8
```

#### anyMatch

该方法传入一个Predciate参数，校验流中元素，至少有一个满足条件，返回true，否则返回false

源码：

```java
boolean anyMatch(Predicate<? super T> predicate);
```

使用示例：

```java
public class StreamTest {
    public static void anyMatchTest(List<String> list){
        System.out.println(list.stream().anyMatch(e -> e.length()>10));
        System.out.println(list.stream().anyMatch(e -> e.length()>8));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        anyMatchTest(list);
    }
}
```

执行结果为：

```text
false
true
```

#### allMatch

该方法同样需要一个Predicate参数，用于校验流中的所有元素，只有全部满足规则才能返回true，只要有一个不满足则返回false

源码：

```java
boolean allMatch(Predicate<? super T> predicate);
```

使用示例：

```java
public class StreamTest {
    public static void allMatchTest(List<String> list){
        System.out.println(list.stream().allMatch(e -> e.length()>1));
        System.out.println(list.stream().allMatch(e -> e.length()>3));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        allMatchTest(list);
    }
}
```

执行结果为：

```text
true
false
```

#### noneMatch

该方法同样需要一个Predicate参数，用于校验流中的所有元素,只有所有元素都不满足规则的情况下返回true，否则返回false

源码

```java
boolean noneMatch(Predicate<? super T> predicate);
```

使用示例：

```java
public class StreamTest {
    public static void noneMatchTest(List<String> list){
        System.out.println(list.stream().noneMatch(e -> e.length()>10));
        System.out.println(list.stream().noneMatch(e -> e.length()>8));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        noneMatchTest(list);
    }
}
```

执行结果为：

```text
true
false
```

#### findFirst

无参数，获取流中的第一元素，若流无序，则可能返回任意一个

源码：

```java
Optional<T> findFirst();
```

使用示例：

```java
public class StreamTest {
    public static void findFirstTest(List<String> list){
        System.out.println(list.stream().parallel().findFirst().get());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        findFirstTest(list);
    }
}
```

执行结果为：

```text
123
```

#### findAny

无参数，获取流中的任意元素

源码：

```java
Optional<T> findAny();
```

使用示例：

```java
public class StreamTest {
    public static void findAnyTest(List<String> list){
        System.out.println(list.stream().parallel().findAny().get());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        findAnyTest(list);
    }
}
```

执行结果为：

```text
asdaa
```