---
title: Java Comparable接口和Comparator接口
date: 2020-03-06 14:16:36
categories: java
tags:
---

### Comparable接口

Comparable是排序接口，字面意义为**可比较的**，位于java.lang包中

此接口强制对于实现它的类的对象进行整体排序，此排序称为该类的自然排序，类的`compareTo`方法称为类的自然排序方法，实现此接口的对象列表（和数组）可以通过 `Collections.sort`（和 `Arrays.sort `）进行自动排序，实现此接口的对象可以用作`SortedMap`（有序映射表）中的键或`SortedSet`（有序集合）中的元素，无需指定比较器 

常见的类如`Integer`, `Double`, `String`都实现了此类

源码

```java
public interface Comparable<T> {
 public int compareTo(T o);
}
```

接口只包含一个方法，比较当前对象与指定对象的顺序

如果当前对象小于、等于、大于指定对象，分别返回负整数、0、正整数

如`x.compareTo(y)`:

x<y，返回负整数；x=y，返回0；x>y，返回正整数

示例

JDK中`Integer`类部分源码

```java
public final class Integer extends Number implements Comparable<Integer> {

    private final int value;
    
    public int compareTo(Integer anotherInteger) {
        return compare(this.value, anotherInteger.value);
    }
    public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
    public static int compareUnsigned(int x, int y) {
        return compare(x + MIN_VALUE, y + MIN_VALUE);
    }
```

可以看到，`compareTo`方法其中调用了`compare`方法，这是JDK1.7增加的方法。在`Integer`中新增这个方法是为了减少不必要的自动装箱拆箱。传入`compare`方法的是两个`Integer`的值value`x`和`y`。
 如果`x < y`， 返回`-1`；如果`x = y`， 返回`0`；如果`x > y`， 返回`1`。

后面的`compareUnsigned`是JDK1.8新加入的方法, 用来比较无符号数

### Comparator接口

比较器接口，位于java.util包中

若需要控制某个类的次序，而该类本身没有实现Comparable接口，不支持排序，那么就可以通过实现Comparator类建立一个该类的比较器实现排序

实现该接口的类可以作为参数传入`Collections.sort()`或`Arrays.sort()`

与Comparable不同的是，Comparator可以自定义排序顺序，而不是强制限定为自然排序

同时Comparable接口将比较代码嵌入需要进行比较的类的自身代码中（**内部比较器**），而Comparator接口在一个独立的类中实现比较（**外部比较器**）

源码

```java
public interface Comparator<T> {
	int compare(T o1, T o2);
	boolean equals(Object obj);
	 //省略其余源码
    //其余源码均为默认default方法和静态static方法
    //实现此接口只需实现面两个方法即可
 
 }
```

示例

[来源](https://www.cnblogs.com/Kevin-mao/p/5912775.html)

```java
package test;

import java.util.Comparator;
/**
 * 具体的比较类（比较器），实现Comparator接口
 * @author breeze
 *
 */
public class ComparatorConsunInfo implements Comparator<ConsumInfo> {
    /**
     * 顺序（从小到大）：
     * if(price < o.price){
            return -1;
        }
        if(price > o.price){
            return 1;
        }
     * 倒序（从大到小）：
     * if(price < o.price){
            return 1;
        }
        if(price > o.price){
            return -1;
        }
     */
    @Override
    public int compare(ConsumInfo o1, ConsumInfo o2) {
         //首先比较price，如果price相同，则比较uid
        if(o1.getPrice() > o2.getPrice()){
            return 1;
        }
        
        if(o1.getPrice() < o2.getPrice()){
            return -1;
        }
        
        if(o1.getPrice() == o2.getPrice()){
            if(o1.getUid() > o2.getUid()){
                return 1;
            }
            if(o1.getUid() < o2.getUid()){
                return -1;
            }
        }
        return 0;
    }

}


/**
 * 需要进行比较的类
 * @author breeze
 *
 */
public class ConsumInfo{
    private int uid;
    private String name;
    private double price;
    private Date datetime;
    
    public ConsumInfo() {
        // TODO Auto-generated constructor stub
    }
    
    public ConsumInfo(int uid,String name,double price,Date datetime){
        this.uid = uid;
        this.name = name;
        this.price = price;
        this.datetime = datetime;
                
    }
    
    //省略set和get

    @Override
    public String toString() {
        return "ConsumInfo [uid=" + uid + ", name=" + name + ", price=" + price
                + ", datetime=" + datetime + "]";
    }
    
}


//测试类
public class ConsumInfoTest {
    
    public static void main(String[] args) {
        
        ConsumInfo consumInfo1 = new ConsumInfo(100, "consumInfo1", 400.0,new Date());
        ConsumInfo consumInfo2 = new ConsumInfo(200, "consumInfo1", 200.0,new Date());
        ConsumInfo consumInfo3 = new ConsumInfo(300, "consumInfo1", 100.0,new Date());
        ConsumInfo consumInfo4 = new ConsumInfo(400, "consumInfo1", 700.0,new Date());
        ConsumInfo consumInfo5 = new ConsumInfo(500, "consumInfo1", 800.0,new Date());
        ConsumInfo consumInfo6 = new ConsumInfo(600, "consumInfo1", 300.0,new Date());
        ConsumInfo consumInfo7 = new ConsumInfo(700, "consumInfo1", 900.0,new Date());
        ConsumInfo consumInfo8 = new ConsumInfo(800, "consumInfo1", 400.0,new Date());
        
        List<ConsumInfo> list = new ArrayList<ConsumInfo>();
        list.add(consumInfo1);
        list.add(consumInfo2);
        list.add(consumInfo3);
        list.add(consumInfo4);
        list.add(consumInfo5);
        list.add(consumInfo6);
        list.add(consumInfo7);
        list.add(consumInfo8);
        
        System.out.println("排序前：");
        //排序前
        for(ConsumInfo consumInfo : list ){
            System.out.println(consumInfo);
        }
        ComparatorConsunInfo comparatorConsunInfo = new ComparatorConsunInfo();//比较器
        Collections.sort(list,comparatorConsunInfo);//排序
        System.out.println("排序后：");
        //排序后
        for(ConsumInfo consumInfo :list){
            System.out.println(consumInfo);
        }
    }
}
```

