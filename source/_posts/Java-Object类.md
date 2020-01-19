---
title: Java-Object类
date: 2020-01-19 09:39:47
categories: java
tags: Object
---

### 概述

------

Object类是JDK中所有类的基类，位于java.lang包中（此包中所有类均无需手动导入，系统会在程序编译期间自动导入），一共有13个方法（包括类构造器），类源码如下：

```java
package java.lang;

public class Object {

    private static native void registerNatives();
    static {
        registerNatives();
    }

    public final native Class<?> getClass();

    public native int hashCode();

    public boolean equals(Object obj) {
        return (this == obj);
    }

    protected native Object clone() throws CloneNotSupportedException;

    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

    public final native void notify();

    public final native void notifyAll();

    public final native void wait(long timeout) throws InterruptedException;

    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && timeout == 0)) {
            timeout++;
        }

        wait(timeout);
    }

    public final void wait() throws InterruptedException {
        wait(0);
    }

    protected void finalize() throws Throwable { }
}
```

### 类中方法及说明

```java
private static native void registerNatives()   //私有方法
public final native Class<?> getClass()    //返回此 Object 的运行类。
public native int hashCode()    //用于获取对象的哈希值。
public boolean equals(Object obj)     //用于确认两个对象是否“相同”。
protected native Object clone()    //创建并返回此对象的一个副本。 
public String toString() toString()   //返回该对象的字符串表示。   
public final native void notify()    //唤醒在此对象监视器上等待的单个线程。   
public final native void notifyAll()     //唤醒在此对象监视器上等待的所有线程。   
public final native void wait(long timeout)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或        者超过指定的时间量前，导致当前线程等待。   
public final void wait(long timeout, int nanos)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量前，导致当前线程等待。
 public final void wait()    //用于让当前线程失去操作权限，当前线程进入等待序列
protected void finalize() finalize()    //当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法，JVM自动调用。
public Object()   //大部分情况下，Java中通过形如 new A(args..)形式创建一个属于该类型的对象。其中A即是类名.A(args..)即此类定义中相对应的构造函数。通过此种形式创建的对象都是通过类中的构造函数完成。为体现此特性，Java中规定：在类定义过程中，对于未定义构造函数的类，默认会有一个无参数的构造函数，作为所有类的基类，Object类自然要反映出此特性，在源码中，未给出Object类构造函数定义，但实际上，此构造函数是存在的。当然，并不是所有的类都是通过此种方式去构建，也自然的，并不是所有的类构造函数都是public。
```

**注意：**`native`关键字，Java中，用native关键字修饰的函数表明该方法的实现并不是在Java中去完成，而是由C/C++去完成，并被编译成了`.dll`，由Java去调用

#### 常用方法说明：

- `String toString()`

  直接使用`toString()`方法时，由源码可知，默认返回的是一个地址编码

  ```java
  package com.company;
  
  /**
   * @author yanzx
   * @version 1.0
   * @date 2020/1/19 10:19
   */
  public class Demo {
      public static void main(String[] args) {
          Object obj=new Object();
          System.out.println(obj.toString());
      }
  }
  ```

  ![image-20200119102401761](Java-Object%E7%B1%BB.assets/image-20200119102401761.png)

  因此，实际使用时需要重写该方法，

  ```java
  package com.company;
  
  /**
   * @author yanzx
   * @version 1.0
   * @date 2020/1/19 10:19
   */
  public class Demo {
      @Override
      public String toString(){
          return ("toString()方法重写");
      }
  
      public static void main(String[] args) {
         Demo demo=new Demo();
         System.out.println(demo.toString());//toString()方法重写
          System.out.println("toString() test"+123);//Object是所有对象的父类，因此不同对象的内容输出都是通过复写toString()方法实现的
      }
  }
  
  ```

  ![image-20200119103248663](Java-Object%E7%B1%BB.assets/image-20200119103248663.png)

- `boolean equals()`

  该方法用于比较两个对象是否相同，实际上是比较两个对象的内存地址是否相同，如下，demo和demo1内容虽然相同，但使用`new`创建了不同的空间，地址不同，比较结果为`false`

  ```java
  package com.company;
  
  import org.jetbrains.annotations.Contract;
  
  /**
   * @author yanzx
   * @version 1.0
   * @date 2020/1/19 10:19
   */
  public class Demo {
  
      private String param;
  
      public Demo(String param){
         this.param=param;
     }
  
      public static void main(String[] args) {
         Demo demo=new Demo("p1");
         Demo demo1=new Demo("p1");
          System.out.println(demo.equals(demo1));
      }
  }
  
  ```

  ![image-20200119104647558](Java-Object%E7%B1%BB.assets/image-20200119104647558.png)

  若想要比较内容，则需复写`equals()`方法，如下，比较结果为`true`

  ```java
  package com.company;
  
  import org.jetbrains.annotations.Contract;
  
  /**
   * @author yanzx
   * @version 1.0
   * @date 2020/1/19 10:19
   */
  public class Demo {
  
      private String param;
  
      public Demo(String param){
         this.param=param;
     }
     @Override
      public boolean equals(Object obj){
          if(obj==null){
              return false;
          }
          //判断地址是否相同，地址相同则是自己在和自己比较
          if(this==obj){
              return true;
          }
          //判断类是否相同，类不同则无法比较
          if(!(obj instanceof Demo)){
              return false;
          }
         /**
          * 传入的对象不为空，对象地址不同而且类相同
          * obj向下转型,比较属性值
          */
         Demo demo=(Demo) obj;
          return demo.param.equals(this.param);
      }
      public static void main(String[] args) {
         Demo demo=new Demo("p1");
         Demo demo1=new Demo("p1");
          System.out.println(demo.equals(demo1));
      }
  }
  
  ```

  ![image-20200119105802037](Java-Object%E7%B1%BB.assets/image-20200119105802037.png)

  **注意：**`euqals()`和`==`的区别

  - `euqals()`用于引用类型间的比较，默认比较的是内存地址值；重写方法后比较的是对象的内容是否相同；继承自`Object`类的类大都对该方法实现了重写，因此使用此方法是大都比较的是值

  - `==`用于基本类型间的比较时，比较的时值；用于引用类型间的比较时比较的是内存地址；

- `int hashCode()`

  返回该对象的哈希值，默认情况下，该方法会根据对象的地址来计算；

  不同对象的`hashCode()`的值一般是是不相同，但同一个对象的`hashCode()`值一定相同

  `hashCode()`定义在Object中，意味着每一个类都包含该方法，但该方法仅在创建**某个类的散列表**时，`hashCode()`才有作用，用于获取对象的散列码进而确定该类的每一个对象在散列表中的位置，其他情况如创建类的单个对象，，创建类的对象数组，类的`hashCode()`作用

  上述散列表是指`Java`集合中本质是散列表的类，如`HashMap`，`HashTable`，`HashSet`

  详见[此文](https://www.cnblogs.com/skywang12345/p/3324958.html)

  ```java
  package com.company;
  
  import org.jetbrains.annotations.Contract;
  
  /**
   * @author yanzx
   * @version 1.0
   * @date 2020/1/19 10:19
   */
  public class Demo {
  
      private String param;
  
      public Demo(String param){
         this.param=param;
     }
  
      public static void main(String[] args) {
         Demo demo=new Demo("p1");
         int h=demo.hashCode();
         Demo demo1=new Demo("p1");
         int h1=demo1.hashCode();
          System.out.println(h);
          System.out.println(h1);
          System.out.println(h==h1);
      }
  }
  
  ```

  ![image-20200119112447827](Java-Object%E7%B1%BB.assets/image-20200119112447827.png)

#### **补充**

`hashCode()`和`equals()`的相关规定：

- 如果两个对象相等，则hashcode一定也是相同的

- 两个对象相等，则对两个对象分别调用`equals()`都返回true

- 两个对象有相同的hashcode值，它们也不一定是相等的

- **因此，`equals()`方法被重写过，则`hashCode()`也必须被重写**

- `hashCode()` 的默认行为是对堆上的对象产生独特值（基于地址）。如果没有重写 `hashCode()`，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

- 严格的数学逻辑表示为： 两个对象相等 <=>  equals()相等=>hashCode()相等。因此，重写equlas()方法必须重写hashCode()方法，以保证此逻辑严格成立，同时可以推理出：hasCode()不相等 => equals（）不相等 <=> 两个对象不相等。

- 可能有人在此产生疑问：既然比较两个对象是否相等的唯一条件（也是充要条件）是equals，那么为什么还要弄出一个hashCode()，并且进行如此约定，弄得这么麻烦？

  **其实，这主要体现在hashCode()方法的作用上，其主要用于增强哈希表的性能。**

  以集合类中，以Set为例，当新加一个对象时，需要判断现有集合中是否已经存在与此对象相等的对象，如果没有hashCode()方法，需要将Set进行一次遍历，并逐一用equals()方法判断两个对象是否相等，此种算法时间复杂度为o(n)。通过借助于hasCode方法，先计算出即将新加入对象的哈希码，然后根据哈希算法计算出此对象的位置，直接判断此位置上是否已有对象即可。（注：Set的底层用的是Map的原理实现）

  在此需要纠正一个理解上的误区：对象的hashCode()返回的不是对象所在的物理内存地址。甚至也不一定是对象的逻辑地址，hashCode()相同的两个对象，不一定相等，换言之，不相等的两个对象，hashCode()返回的哈希码可能相同。

  因此，重写了equals()方法后，需要重写hashCode()方法。此部分参考自[此文](https://www.iteye.com/blog/ihenu-2233249)

以下内容参考[此文](https://www.cnblogs.com/skywang12345/p/3324958.html)

**二者关系：**

- 不会创建**类对应的散列表**

  不会创建类对应的散列表指不会在HashSet，HashTable，HashMap等这些本质是散列表的数据结构中用到该类。例如不会创建该类的HashSet集合

  在这种情况下，该类的`hashCode()`和`equals()`没有任何关系

  这种情况下，`equals()`用于比较该类的两个对象是否相等（重写该方法用于判断内容是否相同），而`hashCode()`没有任何作用，但为了保持逻辑的一致性，约定重写`equals()`时也需要重写`hashCode()`

- 会创建**类对应的散列表**

  在这种情况下，该类的`hashCode()`和`equals()`有关系：

  - 若两个对象相等，那么它们的`hashCode()`值一定是相等的，这里相等是指，`equals()`比较两个对象是返回`true`

  - 若两个对象`hashCode()`相等，它们并不一定相等，即`equal()`返回值不一定为`true`

    因为在散列表中，`hashCode()`相等，即两个键值对的哈希值相等;然而哈希值相等，并不一定能得出键值对相等。补充说一句：“两个不同的键值对，哈希值相等”，这就是哈希冲突。

    **注意：**

    在这种情况下，若要判断两个对象是否相等，除了要覆盖`equals()`，还有覆盖`hashCode()`方法，否则`equals()`无效

    - ​	只重写`equals()`时：

    ```java
    import java.util.*;
    import java.lang.Comparable;
    
    /**
     * @desc 比较equals() 返回true 以及 返回false时， hashCode()的值。
     *
     * @author skywang
     * @emai kuiwu-wang@163.com
     */
    public class ConflictHashCodeTest1{
    
        public static void main(String[] args) {
            // 新建Person对象，
            Person p1 = new Person("eee", 100);
            Person p2 = new Person("eee", 100);
            Person p3 = new Person("aaa", 200);
    
            // 新建HashSet对象 
            HashSet set = new HashSet();
            set.add(p1);
            set.add(p2);
            set.add(p3);
    
            // 比较p1 和 p2， 并打印它们的hashCode()
            System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
            // 打印set
            System.out.printf("set:%s\n", set);
        }
    
        /**
         * @desc Person类。
         */
        private static class Person {
            int age;
            String name;
    
            public Person(String name, int age) {
                this.name = name;
                this.age = age;
            }
    
            public String toString() {
                return "("+name + ", " +age+")";
            }
    
            /** 
             * @desc 覆盖equals方法 
             */  
            @Override
            public boolean equals(Object obj){  
                if(obj == null){  
                    return false;  
                }  
                  
                //如果是同一个对象返回true，反之返回false  
                if(this == obj){  
                    return true;  
                }  
                  
                //判断是否类型相同  
                if(this.getClass() != obj.getClass()){  
                    return false;  
                }  
                  
                Person person = (Person)obj;  
                return name.equals(person.name) && age==person.age;  
            } 
        }
    }
    ```

    运行结果：

    ![image-20200119145518853](Java-Object%E7%B1%BB.assets/image-20200119145518853.png)
    
    发现p1和p2相等，HashSet中有重复元素，这是p1和p2内容虽然相等，但hashcode不等，故HashSet添加元素时，认为它们不相等
    
- ​	`equals()`和`hashCode()`都重写时
    
```java
    import java.util.*;
import java.lang.Comparable;
    
    /**
     * @desc 比较equals() 返回true 以及 返回false时， hashCode()的值。
     *
     * @author skywang
     * @emai kuiwu-wang@163.com
     */
    public class ConflictHashCodeTest2{
    
        public static void main(String[] args) {
            // 新建Person对象，
            Person p1 = new Person("eee", 100);
            Person p2 = new Person("eee", 100);
            Person p3 = new Person("aaa", 200);
            Person p4 = new Person("EEE", 100);
    
            // 新建HashSet对象 
            HashSet set = new HashSet();
            set.add(p1);
            set.add(p2);
            set.add(p3);
            set.add(p4);
    
            // 比较p1 和 p2， 并打印它们的hashCode()
            System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
            // 比较p1 和 p4， 并打印它们的hashCode()
            System.out.printf("p1.equals(p4) : %s; p1(%d) p4(%d)\n", p1.equals(p4), p1.hashCode(), p4.hashCode());
            // 打印set
            System.out.printf("set:%s\n", set);
        }
    
        /**
         * @desc Person类。
         */
        private static class Person {
            int age;
            String name;
    
            public Person(String name, int age) {
                this.name = name;
                this.age = age;
            }
    
            public String toString() {
                return name + " - " +age;
            }
    
            /** 
             * @desc重写hashCode 
             */  
            @Override
            public int hashCode(){  
                int nameHash =  name.toUpperCase().hashCode();
                return nameHash ^ age;
            }
    
            /** 
             * @desc 覆盖equals方法 
             */  
            @Override
            public boolean equals(Object obj){  
                if(obj == null){  
                    return false;  
                }  
                  
                //如果是同一个对象返回true，反之返回false  
                if(this == obj){  
                    return true;  
                }  
                  
                //判断是否类型相同  
                if(this.getClass() != obj.getClass()){  
                    return false;  
                }  
                  
                Person person = (Person)obj;  
                return name.equals(person.name) && age==person.age;  
            } 
        }
    }
    ```
    
    运行结果：
    
```java
    //Person p1 = new Person("eee", 100);
//Person p2 = new Person("eee", 100);
    //Person p3 = new Person("aaa", 200);
    //Person p4 = new Person("EEE", 100);
    ```
    
    ![image-20200119145215061](Java-Object%E7%B1%BB.assets/image-20200119145215061.png)
    
 这下，equals()生效了，HashSet中没有重复元素。
    
比较p1和p2，我们发现：它们的`hashCode()`相等，通过equals()比较它们也返回true。所以，p1和p2被视为相等。
    
 比较p1和p4，我们发现：虽然它们的`hashCode()`相等；但是，通过equals()比较它们返回false。所以，p1和p4被视为不相等。



