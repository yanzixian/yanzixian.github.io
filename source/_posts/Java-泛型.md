---
title: Java 泛型
categories: Java
tags: generic
---

#### 概述

泛型，即”参数化类型“，将类型有原来的具体类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（也可以称为类型形参），在使用/调用时传入具体的类型（类型实参）。泛型的使用有泛型类，泛型接口，泛型方法

#### 泛型类

定义一个泛型类：`public class GenericClass<T> {......}`

```java
//此处T可以随便写为任意标识，常见的如T,E,K,V等形式
//实例化泛型类时，必须指定T的具体类型
public class GenericClass<T>{
	//key 的类型为T，由外部指定
	private T key;
	//构造方法形参类型为T，由外部指定
	public GenericClass( T key){
		this.key=key;
	}
	//成员方法的返回值类型为T，有外部指定
	public T getKey(){
	return this.key;
	}
}
```

```java
//实例化时指定具体类型
//泛型的类型参数只能是类类型（包括自定义类），不能时简单类型，如Integer，不能是int
//传入的实参类型需与泛型的类型参数类型相同，即Integer<=>123
GenericClass<Integer> genericInt=new GenericClass<Integer>(123);
//传入的实参类型需与泛型的类型参数类型相同,即String<=>"123"
GenericClass<String> genericStr=new GenericClass<String>("123")
```

#### 泛型接口

定义一个泛型接口：`public interface GenericInterface<T>{.....}`

```java
public interface GenericInterface<T>{
	T getData();
}

```

##### 泛型接口的实现

实现方法一

```java
//实现泛型接口的类，未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需要将泛型的声明也一起加入到类中
public ImlGenericInterface1<T> implements GenericInterface<T>{
	private T data;
	public void setData(T data){
		this.data=data;
	}
	@override
	public T getData(){
		return data;
	}
	public static void main(String[] args) {
        ImplGenericInterface1<String> implGenericInterface1 = new ImplGenericInterface1<>();
        implGenericInterface1.setData("Generic Interface1");
        System.out.println(implGenericInterface1.getData());
    }
}
```

实现方法二

```java
//实现泛型接口的类，传入泛型实参时，则所有使用泛型的地方都要替换成传入的实参类型
public class ImplGenericInterface2 implements GenericInterface<String> {
    @Override
    public String getData() {
        return "Generic Interface2";
    }

    public static void main(String[] args) {
        ImplGenericInterface2 implGenericInterface2 = new ImplGenericInterface2();
        System.out.println(implGenericInterface2.getData());
    }
}
```

#### 泛型方法

泛型方法的定义：`public <T> T genericMethods(T a,T b){......}`，调用方法时指明泛型的具体类型

```java
public class ExampleClass {
    private static int add(int a, int b) {
        System.out.println(a + "+" + b + "=" + (a + b));
        return a + b;
    }

    private static <T> T genericAdd(T a, T b) {
        System.out.println(a + "+" + b + "="+a+b);
        return a;
    }

    public static void main(String[] args) {
        GenericMethod1.add(1, 2);
        GenericMethod1.<String>genericAdd("a", "b");
    }
}
```



```java
public class ExampleClass1 {
	//内部类
    static class Animal {
        @Override
        public String toString() {
            return "Animal";
        }
    }

    static class Dog extends Animal {
        @Override
        public String toString() {
            return "Dog";
        }
    }

    static class Fruit {
        @Override
        public String toString() {
            return "Fruit";
        }
    }

    static class GenericClass<T> {

        public void show01(T t) {
            System.out.println(t.toString());
        }

        public <T> void show02(T t) {
            System.out.println(t.toString());
        }

        public <K> void show03(K k) {
            System.out.println(k.toString());
        }
    }

    public static void main(String[] args) {
        Animal animal = new Animal();
        Dog dog = new Dog();
        Fruit fruit = new Fruit();
        GenericClass<Animal> genericClass = new GenericClass<>();
        //泛型类在初始化时限制了参数类型
        genericClass.show01(dog);
//        genericClass.show01(fruit);

        //泛型方法的参数类型在使用时指定
        genericClass.show02(dog);
        genericClass.show02(fruit);

        genericClass.<Animal>show03(animal);
        genericClass.<Animal>show03(dog);
        genericClass.show03(fruit);
//        genericClass.<Dog>show03(animal);
    }
}
```

泛型方法总结

```java
public class GenericTest {
   //这个类是个泛型类，在上面已经介绍过
  public class Generic<T>{     
        private T key;   
	public Generic(T key) {
        this.key = key;
    }

    //我想说的其实是这个，虽然在方法中使用了泛型，但是这并不是一个泛型方法。
    //这只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
    //所以在这个方法中才可以继续使用 T 这个泛型。
    public T getKey(){
        return key;
    }

    /**
     * 这个方法显然是有问题的，在编译器会给我们提示这样的错误信息"cannot reslove symbol E"
     * 因为在类的声明中并未声明泛型E，所以在使用E做形参和返回值类型时，编译器会无法识别。
    public E setKey(E key){
         this.key = keu
    }
    */
}

/** 
 * 这才是一个真正的泛型方法。
 * 首先在public与返回值之间的<T>必不可少，这表明这是一个泛型方法，并且声明了一个泛型T
 * 这个T可以出现在这个泛型方法的任意位置.
 * 泛型的数量也可以为任意多个 
 *    如：public <T,K> K showKeyName(Generic<T> container){
 *        ...
 *        }
 */
public <T> T showKeyName(Generic<T> container){
    System.out.println("container key :" + container.getKey());
    //当然这个例子举的不太合适，只是为了说明泛型方法的特性。
    T test = container.getKey();
    return test;
}

//这也不是一个泛型方法，这就是一个普通的方法，只是使用了Generic<Number>这个泛型类做形参而已。
public void showKeyValue1(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}

//这也不是一个泛型方法，这也是一个普通的方法，只不过使用了泛型通配符?

public void showKeyValue2(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}

 /**
 * 这个方法是有问题的，编译器会为我们提示错误信息："UnKnown class 'E' "
 * 虽然我们声明了<T>,也表明了这是一个可以处理泛型的类型的泛型方法。
 * 但是只声明了泛型类型T，并未声明泛型类型E，因此编译器并不知道该如何处理E这个类型。
public <T> T showKeyName(Generic<E> container){
    ...
}  
*/

/**
 * 这个方法也是有问题的，编译器会为我们提示错误信息："UnKnown class 'T' "
 * 对于编译器来说T这个类型并未项目中声明过，因此编译也不知道该如何编译这个类。
 * 所以这也不是一个正确的泛型方法声明。
public void showkey(T genericObj){

}
*/

public static void main(String[] args) {
}
}
```

#### 泛型类型的限定

##### 对类的限定

`public class TypeLimitForClass<T extends List&Serializable>{}`

```java
//T限定未List和Serializable
public class TypeLimitForClass<T extends List & Serializable> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public static void main(String[] args) {
        ArrayList<String> stringArrayList = new ArrayList<>();
        stringArrayList.add("A");
        stringArrayList.add("B");
        ArrayList<Integer> integerArrayList = new ArrayList<>();
        integerArrayList.add(1);
        integerArrayList.add(2);
        integerArrayList.add(3);
        //T具体为ArrayList
        TypeLimitForClass<ArrayList> typeLimitForClass01 = new TypeLimitForClass<>();
        typeLimitForClass01.setData(stringArrayList);
        TypeLimitForClass<ArrayList> typeLimitForClass02 = new TypeLimitForClass<>();
        typeLimitForClass02.setData(integerArrayList);

        System.out.println(getMinListSize(typeLimitForClass01.getData().size(), typeLimitForClass02.getData().size()));

    }

    public static <T extends Comparable<T>> T getMinListSize(T a, T b) {
        return (a.compareTo(b) < 0) ? a : b;
    }
```

##### 对方法的限定

`public static <T extends Comparable<T>> T getMin(T a,T b){}`

```java
public class TypeLimitForMethod {

    /**
     * 计算最小值
     * 如果要实现这样的功能就需要对泛型方法的类型做出限定
     */
//    private static <T> T getMin(T a, T b) {
//        return (a.compareTo(b) > 0) ? a : b;
//    }

    /**
     * 限定类型使用extends关键字指定
     * 可以使类，接口，类放在前面接口放在后面用&符号分割
     * 例如：<T extends ArrayList & Comparable<T> & Serializable>
     */
    public static <T extends Comparable<T>> T getMin(T a, T b) {
        return (a.compareTo(b) < 0) ? a : b;
    }

    public static void main(String[] args) {
        System.out.println(TypeLimitForMethod.getMin(2, 4));
        System.out.println(TypeLimitForMethod.getMin("a", "r"));
    }
}
```

#### 泛型类型通配符

类型通配符一般是使用`？`代替具体的类型实参

###### 注意

此处`？`是***类型实参***，不是类型形参，是一种真实的类型，可以看作所有类型的父亲

我们知道`Ingeter`是`Number`的一个子类，同时在特性章节中我们也验证过`Generic<Integer>`与`Generic<Number>`实际上是相同的一种基本类型。那么问题来了，在使用`Generic<Number>`作为形参的方法中，能否使用`Generic<Integer>`的实例传入呢？在逻辑上类似于`Generic<Number>`和`Generic<Integer>`是否可以看成具有父子关系的泛型类型呢？

为了弄清楚这个问题，我们使用`Generic<T>`这个泛型类继续看下面的例子：

```java
public void showKeyValue(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```

```java
Generic<Integer> gInteger = new Generic<Integer>(123);
Generic<Number> gNumber = new Generic<Number>(456);

showKeyValue(gNumber);

// showKeyValue这个方法编译器会为我们报错：Generic<java.lang.Integer> 
// cannot be applied to Generic<java.lang.Number>
// showKeyValue(gInteger);
```

通过提示信息我们可以看到`Generic<Integer>`不能被看作为``Generic<Number>`的子类。由此可以看出:同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

回到上面的例子，如何解决上面的问题？总不能为了定义一个新的方法来处理`Generic<Integer>`类型的类，这显然与java中的多台理念相违背。因此我们需要一个在逻辑上可以表示同时是`Generic<Number>`和`Generic<Integer>`父类的引用类型。由此类型通配符应运而生。

```java
public void showKeyValue1(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```