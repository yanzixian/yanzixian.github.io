---
title: Java-Predicate
date: 2020-03-02 10:26:14
categories: java
tags:
---

### Java Predicate接口

#### 概述

Predicate同Funciton一样，也是Java 8引入的函数式接口，主要作用就是提供一个test方法，接受一个参数并返回一个布尔类型

源码：

```java
@FunctionalInterface
public interface Predicate<T> {
    //对于传入的参数进行断言测试，满足断言返回true,否则返回false
 	boolean test(T t);
    //接受一个Predicate参数，与当前断言以且（and）的关系过滤数据，如果当前断言结果为false，则不会评估另一断言
    //返回结果为一个断言
  	default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    //返会与当前断言逻辑为反的断言
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    //接受一个Predicate参数，与当前断言以或（or）的关系过滤数据，如果当前断言结果为true，则不会评估另一断言
    //返回结果为一个断言
 	default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
    //静态方法，接受一个Predicate参数，测试两个参数是否相等
    //返回一个断言
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}

```

#### 使用

[示例来源](https://www.jianshu.com/p/ea429fe9f2f3)

```java
int[] numbers= {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15};
        List<Integer> list=new ArrayList<>();
        for(int i:numbers) {
            list.add(i);
        }
        Predicate<Integer> p1=i->i>5;
        Predicate<Integer> p2=i->i<20;
        Predicate<Integer> p3=i->i%2==0;
        List test=list.stream().filter(p1.and(p2).and(p3)).collect(Collectors.toList());
        System.out.println(test.toString());
/** print:[6, 8, 10, 12, 14]*/
```

定义了3个断言p1，p2，p3，对于一个list中的数字，过滤出所有大于5，小于20，并且是偶数的列表

更改filter中的条件可实现不同的过滤，如：过滤出所有大于5，小于20，并且是奇数的列表，使用negate()取反即可

```java
List test=list.stream().filter(p1.and(p2).and(p3.negate())).collect(Collectors.toList());
/** print:[7, 9, 11, 13, 15]*/
```

特别地，isEqual方法可以当做==操作符来使用

```java
List test=list.stream().filter(p1.and(p2).and(p3.negate()).and(Predicate.isEqual(7)))
            .collect(Collectors.toList());
/** print:[7] */
```

