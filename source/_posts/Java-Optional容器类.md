---
title: Java Optional容器类
date: 2020-02-24 13:59:10
categories: java
tags:
---

#### Java 8 Optional 类

Optional类是一个可以为`null`的容器对象，可以保存类型T的值，或者仅仅保存`null`

该类主要是为了解决空指针异常（NullPointerException）问题而引入，可以不免显式的`null`值检查

[参见](https://www.cnblogs.com/zhangboyu/p/7580262.html)

Java 8之前，任何访问对象方法和属性的操作都可能导致空指针异常，例如

```java
String isocode = user.getAddress().getCountry().getIsocode().toUpperCase();
```

为了不触发异常，需要进行检查，检查的步骤十分繁琐：

```java
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        Country country = address.getCountry();
        if (country != null) {
            String isocode = country.getIsocode();
            if (isocode != null) {
                isocode = isocode.toUpperCase();
            }
        }
    }
}
```

而Optional类很好的解决了这个问题

##### 创建Optional实例

Optional类的部分源码：

```java
/**
     * Common instance for {@code empty()}.
     */
    private static final Optional<?> EMPTY = new Optional<>();

    /**
     * If non-null, the value; if null, indicates no value is present
     */
    private final T value;

    /**
     * Constructs an empty instance.
     *
     * @implNote Generally only one empty instance, {@link Optional#EMPTY},
     * should exist per VM.
     */
    private Optional() {
        this.value = null;
    }

	private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

 	public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
	 public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
 	public static <T> Optional<T> ofNullable(T value) 	{
        return value == null ? empty() : of(value);
    }
```

可以看出Optional类的两个构造方法都是`private`，无法在类外通过`new`的方法创建一个对象实例，但Optional类提供了`empty()`，`of(T value)`，`ofNullable(T value)`三个`public`的`static`方法来创建对象：

```java
//创建一个包装对象值为空的Optional对象
Optional<String> str1=Optional.empty();
//创建一个包装对象值非空（此处为"str2"）的Optional对象
Optional<String> str2=Optional.of("str2");
//创建一个包装对象值允许为空的Optional对象，在不确定包装对象值是否为空时，需使用此方法包装对象来创建Optional对象
Optional<String> str3=Optional.ofNullable(null);
```

##### Optional类主要方法：

###### get()

用于获取Optional所包装对象的实际值，如果为空，则会抛出NoSuchElementException异常

```java
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```

###### isPresent()

判断包装对象值是否为空，非空返回true

```java
public boolean isPresent() {
    return value != null;
}
```

###### ifPresent()

此方法接受一个Consumer对象，如果包装对象值非空，则执行Consumer的accept()方法

```java
public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
        consumer.accept(value);
}
```

示例：[来源](https://www.jianshu.com/p/d81a5f7c9c4e)

```java
public static void printName(Student student)
{
    Optional.ofNullable(student).ifPresent(u ->System.out.println("The student name is : " + u.getName()));
    }
```

上述示例用于打印学生姓名，由于ifPresent()方法内部做了null值检查，调用前无需担心NPE问题

###### filter()

```java
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent())
        return this;
    else
        return predicate.test(value) ? this : empty();
}
```

filter()方法接受参数为Predicate对象，用于对Optional对象进行过滤，如果符合Predicate的条件，返回Optional对象本身，否则返回一个空的Optional对象。举例如下：[来源](https://www.jianshu.com/p/d81a5f7c9c4e)

```csharp
 public static void filterAge(Student student)
    {
        Optional.ofNullable(student).filter( u -> u.getAge() > 18).ifPresent(u ->  System.out.println("The student age is more than 18."));
    }
```

上述示例中，实现了年龄大于18的学生的筛选

###### map()

此方法接受一个函数对象参数，如果包装对象值为空，返回一个空的Optional对象

若对象值非空，使用Function函数对包装值进行计算，并返回一个计算后的新的Optional对象（包装对象类型可能会发送改变）

```java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Optional.ofNullable(mapper.apply(value));
    }
}
```

示例：[来源](https://www.jianshu.com/p/d81a5f7c9c4e)

```java
   public static Optional<Integer> getAge(Student student)
    {
        return Optional.ofNullable(student).map(u -> u.getAge()); 
    }
```

上述代码中，先用ofNullable()方法构造一个Optional<Student>对象，然后用map()计算学生的年龄，返回Optional<Integer>对象（如果student为null, 返回map()方法返回一个空的Optional对象）

###### flatMap()

此方法与Map()类似，不同之处在于Function对象的返回值已经为Optional对象，不需要再进行包装

```java
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}
```

[示例：](https://www.jianshu.com/p/d81a5f7c9c4e)

```java
  public static Optional<Integer> getAge(Student student)
    {
        return Optional.ofNullable(student).flatMap(u -> Optional.ofNullable(u.getAge())); 
    }
```

###### orElse()

若包装对象值非空，返回包装对象值，否则返回参数other的值

**注意：**返回的是包装对象值，不是Optional对象

```java
public T orElse(T other) {
    return value != null ? value : other;
}
```

###### orElseGet()

与orElse()方法类似，区别在于orElseGet()方法的入参为一个Supplier对象，用Supplier对象的get()方法的返回值作为默认值

```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```

###### orElseThrow()

与orElseGet()方法相似，入参都是Supplier对象，只不过orElseThrow()的Supplier对象必须返回一个Throwable异常，并在orElseThrow()中将异常抛出,适用于包装对象值为空时需要抛出特定异常的场景

```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
        return value;
    } else {
        throw exceptionSupplier.get();
    }
}
```