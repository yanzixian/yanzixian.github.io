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