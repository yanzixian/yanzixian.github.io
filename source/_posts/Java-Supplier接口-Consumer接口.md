---
title: Java Supplier接口-Consumer接口
date: 2020-03-02 14:05:16
categories: java
tags:
---

### Java Supplier接口

#### 概述

Supplier接口是Java 8引入的新的函数式接口，源码注释解释为**Represents a supplier of results**，结果的提供者，并且不要求每一次调用Supplier都提供一个新的或不同的结果，一般来讲就是用来创建对象的，但不同于传统的使用new来创建对象的语法；源码也十分简单

源码

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

#### 使用

[示例来源](https://www.jianshu.com/p/7736ee4f8d36)

```java
public class SupplierTest{
    
    String name = "小明";
    
    SupplierTest(){
        System.out.println(name);
    }

    public static void main(String[] args) {

        // 创建Supplier容器，声明为TestSupplier类型,此时并不会调用对象的构造方法，即不会创建对象
        Supplier<SupplierTest> supplier = SupplierTest::new;
        System.out.println("get() 调用前...");
        // 调用get()方法时,才调用对象的构造方法，创建对象实列
        supplier.get();
        System.out.println("get() 调用后...");
        System.out.println(supplier);
    }
}
/**
get() 调用前...
小明
get() 调用后...
/
```

### Java Consumer接口

#### 概述

Consumer接口时java 8引入的函数式接口，表示一个接受单个输入参数且不返回结果的操作。与大多数其他功能接口不同，Consumer可以通过副作用进行操作，结果的消费着，消费参数对象但不返回值

源码

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    /**
     * Returns a composed {@code Consumer} that performs, in sequence
     先执行当前调用者的accept方法，再执行传入参数的accept方法
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

```

[示例来源](https://www.jianshu.com/p/7736ee4f8d36)

```java
public class CustomerTest {
    
    public static void main(String[] args) {

        // 创建两个容器对象
        Consumer<String> consumer1 = s -> System.out.println("hello 1 :" + s);
        Consumer<String> consumer2 = s -> System.out.println("hello 2 :" + s);

        //  先执行 consumer2.accept() , 如果没有异常再执行 consumer1.accept()
        consumer2.andThen(consumer1).accept("consumer");
    }
}
/**
*hello 2 :consumer
*hello 1 :consumer
/
```

