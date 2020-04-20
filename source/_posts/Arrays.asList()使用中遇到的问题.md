---
title: Arrays.asList()使用中遇到的问题
date: 2020-04-17 14:08:37
categories:java
tags:
---

#### 概述

`Arrays`是java.util包中提供的工具类，主要用于对数组进行各种操作

`Arrays.asList()`是该类中提供的一个静态方法，根据源码注释，作用为基于特定数组返回一个固定大小的列表

> Returns a fixed-size list backed by the specified array

#### 问题

运行如下代码时出现异常：

```java
String[] strs={"str1","str2","str3","str4"};

List<String> arr=Arrays.asList(strs);
        
arr.add("str5");//抛出异常java.lang.UnsupportedOperationException
arr.remove("str1");//抛出异常java.lang.UnsupportedOperationException

System.out.println(arr);
```

这里通过Arrays.asList()将String数组转化为List，但对List进行add和remove操作时却报异常`UnsupportedOperationException`，这是为啥呢，为什么操作不支持呢？

根据上述概述，该方法返回一个`fixed-size`，即大小不能改变的列表，该方法的源码很简单，

```java
 public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```

由源码可知，该方法返回一个`ArrayList`，那为啥`ArrayList`就不支持add和remove操作了呢？

原来，这里的`ArrayList`并不是java.util包中的集合类`ArrayList`，而是在`Arrays`类中自定义的一个内部类，其类的定义源码就在`asList()`方法源码下；

![](https://gitee.com/yanzixian/picBed/raw/master/img202004/20200420144009.png)

![](https://gitee.com/yanzixian/picBed/raw/master/img202004/20200420144148.png)

（话说为啥不换个名）

该内部类源码为

```java
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
    {
    //内部源码省略
    ....
    ....
    }
```

可以看到该类继承了`AbstractList`类，并实现了`RandomAccess`和`Serializable`接口；该类源码中没有定义add，remove方法

进一步查看`AbstractList`类源码

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    //部分源码省略
    ...
    ...
    
 	public boolean add(E e) {
        add(size(), e);
        return true;
    }
    public void add(int index, E element) {
        throw new UnsupportedOperationException();
    }
     public E remove(int index) {
        throw new UnsupportedOperationException();
    }
    
    //部分源码省略
    ...
    ...
}
```

可以看到，该类继承`AbstraceCollection`类并实现`List`接口，执行add，remove操作时均抛出`UnsupportedOperationException`异常；

事实上，查看`AbstraceList`类其余源码可以看到，其中set方法也抛出`UnsupportedOperationException`异常，但ArrayList内部类重写了set方法，所以可以执行set操作

#### 注意：

1. 执行set操作时，如下代码，分别对数组和由数组转化的List集合元素进行修改

 ```java
  public static void main(String[] args) {
      String[] strs={"str1","str2","str3","str4"};
      List<String> list=Arrays.asList(strs);
      
      System.out.println(Arrays.toString(strs));
      System.out.println(list);
      
      strs[0]="str1_new";//对strs进行修改
      list.set(1,"str2_new");//对list进行修改
      
      System.out.println(Arrays.toString(strs));
      System.out.println(list);
  }
 ```

结果如下：

![image-20200420151850155](https://gitee.com/yanzixian/picBed/raw/master/img202004/20200420151851.png)

发现对数组进行的修改影响到了集合，对于集合进行的修改也影响到了数组；

这是因为`asList()`转化得到的集合元素直接引用了数组，数组和集合的变化会同步，这同时可能会导致一些编码上的问题

2. `asList()`方法不可操作基本数据类型；因为由于通过new创建一个Arrays.ArrayList时，其参数为可变长泛型，而基本类型是无法泛型化的

