---
title: Java Function接口
date: 2020-02-26 15:00:05
categories: java
tags:
---

### Function接口

#### 概述

Function接口时java 1.8添加的一个新的特性，使用`@FunctionalInterface`进行标注，表示是函数式接口，具体来说，所有标注了该注解的接口都将能用在lambda表达式上

类源码：

```java
@FunctionalInterface
//接受T对象，返回R对象
public interface Function<T, R> {
  //对参数应用此函数
  R apply(T t);
    //返回一个组合函数，该函数首先将before函数应用于其输入，然后将此函数应用与结果
  default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    //返回一个组合函数，该函数首先将此函数应用于*输入，然后将 after函数应用于结果
  default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    //返回始终返回其输入参数的函数
  static <T> Function<T, T> identity() {
        return t -> t;
    }
}

```

#### 使用

##### apply()

Function是一个泛型接口，其中定义了两个泛型参数T和R，在Function中，T代表输入参数，R代表返回的结果；

Function中没有具体的操作，具体的操作需要我们去为它指定，因此`apply()`具体返回的结果取决于传入的lambda表达式，例如：

```java
public void test(){
	Function<Integet,Integer> test=i->i+1;
	test.apply(1)
}
//结果为2
```

用lambda表达式定义了一个自增的行为，对于参数1执行`apply`,最后返回2

再[例如](https://www.cnblogs.com/rever/p/9725173.html)：

```java
public void test(){
    //定义不同的函数
    Function<Integer,Integer> test1=i->i+1;
    Function<Integer,Integer> test2=i->i*i;
    System.out.println(calculate(test1,5));
    System.out.println(calculate(test2,5));
}
//
public static Integer calculate(Function<Integer,Integer> test,Integer number){
    return test.apply(number);
}
/** print:6*/
/** print:25*/
```

通过定义不同的Function，并传入，实现了在同一个方法中实现不同的操作，称为逻辑的复用

实际使用中可能需要多个函数对于一个参数先后进行操作，[例如：](https://www.cnblogs.com/rever/p/9725173.html)

```java
public void test(){
    Function<Integer,Integer> A=i->i+1;
    Function<Integer,Integer> B=i->i*i;
    //先使用A对参数进行处理，在使用B对A处理的结果再进行处理
    System.out.println("F1:"+B.apply(A.apply(5)));
    //先使用B对参数进行处理，在使用A对B处理的结果再进行处理
    System.out.println("F2:"+A.apply(B.apply(5)));
}
/** F1:36 */
/** F2:26 */
```

##### compose和andThen

为了简化多个连续的函数操作，提供了这两种方法，返回的均为一个Function

如上面源码，compose接受一个Fnuncton参数，先执行传入参数的apply，再执行本身的apply；andThen则执行顺序相反，即

```java
B.apply(A.apply(5));
//等价于
B.compose(A).apply(5);
//或
A.andThen(B).apply(5);

A.apply(B.apply(5));
//等价于
B.andThen(A).apply(5);
//或
A.compose(B).apply(5);
```

