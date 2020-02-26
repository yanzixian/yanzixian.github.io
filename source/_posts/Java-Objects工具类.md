---
title: Java Objects工具类
date: 2020-02-26 10:52:58
categories: java
tags:
---

### Objects

该类是在Java 1.7中提供的一个工具类，为final，不可被继承，提供的方法为static；

该类主要提供了一些对对象进行操作的工具方法，例如计算hash值，返回一个对象的string和对象间的比较等；

类部分源码如下：

```java
public final class Objects {
    private Objects() {
        throw new AssertionError("No java.util.Objects instances for you!");
    }
    ...
    ...
    ...
}
```

作为一个final类型的工具类，无法被继承，却提供了一个private的构造方法（方法体中抛出一个异常），意味着无法在类外创建对象；但在类中各方法也并没有调用该构造方法，理论上该构造方法方法体为空也行，这是，，，程序员的恶趣味吗。。。。。。

#### 工具方法：

##### equals()

- 判断两个对象是否相等：同一个对象或值相等
- 两个对象均为`null`时返回`true`

```java
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```

##### deepEquals()

此方法的比较较为严格

1. 首先，判断两个对象是否是同一个对象（内存地址是否相等）
2. 其次，有一个对象为`null`是返回`false`
3. 最后，提供了两个对象是数组时的比较
   1. 数组比较中首先判断两个对象是否是矩阵，对于矩阵的每个元素，它们是否是同一个对象
   2. 最后调用对象的equals方法（比较两个对象的值）

```java
public static boolean deepEquals(Object a, Object b) {
    if (a == b)
        return true;
    else if (a == null || b == null)
        return false;
    else
        return Arrays.deepEquals0(a, b);
}
```

##### hashCode()

返回对象的hasdCode，若对象为null，返回0

```java
public static int hashCode(Object o) {
    return o != null ? o.hashCode() : 0;
}
```

##### hash()

计算一系列输入值的hash值，输入的一系列值被当成数组，并使用Arrays工具类中的hashCode()计算hash值

```
public static int hash(Object... values) {
    return Arrays.hashCode(values);
}
```

##### toString()

此方法有2种重载方法:

返回对象的String值，对象为`null`时返回“null"字符串

```java
public static String toString(Object o) {
    return String.valueOf(o);
}
```

返回一个对象的String值，对象为空时返回传入的默认字符串

```java
public static String toString(Object o, String nullDefault) {
    return (o != null) ? o.toString() : nullDefault;
}
```

##### compare()

比较两个参数对象

若参数a，b相同，返回0

若a，b均为null，返回0

若a，b不同，使用比较器c进行比较，返回比较后的值

```java
public static <T> int compare(T a, T b, Comparator<? super T> c) {
    return (a == b) ? 0 :  c.compare(a, b);
}
```

##### requireNoNull()

此方法有两个重载方法：

判断对象是否为空，非空时返回原对象；为空时抛出空指针异常

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

判断对象是否为空，非空时返回原对象；为空时抛出具有特定信息的空指针异常

```java
public static <T> T requireNonNull(T obj, String message) {
    if (obj == null)
        throw new NullPointerException(message);
    return obj;
}
```

判断对象是否为空，非空时返回原对象；为空时抛出具有特定信息的空指针异常，使用Supplier类来提供具体信息

异常信息的创建在null检测后进行，因此非null情况下效率优于直接使用string进行异常信息创建

```java
public static <T> T requireNonNull(T obj, Supplier<String> messageSupplier) {
    if (obj == null)
        throw new NullPointerException(messageSupplier.get());
    return obj;
}
```

##### isNull()

对象为空时返回true，否则返回false

```java
public static boolean isNull(Object obj) {
    return obj == null;
}
```

##### nonNull()

对象不为空时返回true，否则返回false

```java
public static boolean nonNull(Object obj) {
    return obj != null;
}
```