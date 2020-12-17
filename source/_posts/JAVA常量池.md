### 前言

此文并非原创，主要是自己学习的总结，以加深理解和记忆，引用了一些大佬的原文,原文地址在文末,感谢大佬

![](https://gitee.com/yanzixian/picBed/raw/master/img202003/感谢大佬.png)

### JAVA常量池  
---
 了解常量池前，需要对于Java虚拟机`JVM`有一个大概的了解

#### JVM内存分布

大多数JVM的内存分布如下图：

[图源]([https://blog.csdn.net/u012988901/article/details/99740694#JVM%E6%98%AF%E4%BB%80%E4%B9%88](https://blog.csdn.net/u012988901/article/details/99740694#JVM是什么))

![20190820101038234](https://gitee.com/yanzixian/picBed/raw/master/img202005/20200513145016.png)

- > **虚拟机栈**：线程私有，生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同到都会创建一个栈帧（Stack Frame）用于存储局部量表、操作数栈、动态链接、方法出口等信息；
  >
  > 局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、long、float、double）、对象引用（reference类型，它不等同于对象本身，可能是一个指向对象始地址的引用指针，也可能是指向一个代表对象的句柄或其地与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址)

- > **本地方法栈**：与虚拟机栈非常相似，也是线程私有的，它们的区别不过是虚拟机栈执行的是Java方法（也就是字节码），而本地方法栈用到的是Native方法

- > **程序计数器**：线程私有的，它可以**看作是当前线程所执行的字节码的行号指示器**

- > **方法区**：各个线程共享的内存区域，它用于存储虚拟机加载的：类信息＋普通常量＋静态常量＋编译器编译后的代码等等；部分存储的是运行时必须的类相关信息，装载进此区域的数据是不会被垃圾收集器回收的，只有关闭Jvm才会释放这块区域占用的内存；
  >
  > 对于Hotspot虚拟机，很多开发者习惯将方法区称之为“永久代（Parmanent Gen)"，但严格本质上说两者不同，或者说使用永久代来实现方法区而己，方法区是一个逻辑上的概念，永久代是方法区（相当于是一个接口interface）的一个实现
  >
  > jdkl.7的版本中，己经将原本放在永久代的字符串常量池移入堆中
  >
  > Jdk1.7中方法区是用永久代实现的，到1.8中是用元空间（MetaSpace）实现的，而元空间使用的是直接内存

**直接内存：**直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域



**注：**jdk1.8中jvm内存分布

大佬的原图

![v2-106a5a2c1293cdf6bf6bebbf7dffeed5_1440w_meitu_1](https://gitee.com/yanzixian/picBed/raw/master/img202005/20200513135219.jpg)

可以看到jdk1.8中，方法区由元空间（使用直接内存）实现；字符串常量池、运行时常量池均在堆中

但也有其他说法认为：

> JDK1.7中永久代逻辑上属于方法区，实际物理上还是放在堆区内，所以方法区内的运行时常量池、类信息都在堆内
>
> JDK1.8中方法区实现改成了元空间，直接放到了直接内存，但是**运行时常量池**物理上还是在堆区，虽然物理存放的地方变了，逻辑上运行时常量池还是属于方法区



---



在 JAVA 语言中有8种基本类型和一种比较特殊的类型`String`。这些类型为了使他们在运行过程中速度更快，更节省内存，都提供了一种常量池的概念

常量池就类似一个JAVA系统级别提供的缓存

常量池大体可以分为两类:**静态常量池**、**运行时常量池**；

通常说的常量池都是运行时常量池

#### 静态常量池

> 存在于`.class`文件中的常量池，class文件中的常量池不仅仅包含字符串(数字)字面量，还包含类、方法的信息，占用class文件绝大部分空间。这种常量池主要用于存放两大类常量：**字面量**(Literal)和**符号引用量**(Symbolic References)，字面量相当于Java语言层面常量的概念，如文本字符串，声明为final的常量值等，符号引用则属于编译原理方面的概念，包括了如下三种类型的常量：
>
> - 类和接口的全限定名
> - 字段名称和描述符
> - 方法名称和描述符

#### 运行时常量池

> jvm虚拟机在完成类装载操作后，将class文件中的常量池载入到内存中，并保存在**方法区**中，我们常说的常量池，就是指方法区中的运行时常量池。
>
> 运行时常量池相对于Class文件常量池的另外一个重要特征是**具备动态性**，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入Class文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用比较多的就是**String类的intern()**方法。
>
> String的intern()方法会查找在常量池中是否存在一份equal相等的字符串,如果有则返回该字符串的引用,如果没有则添加自己的字符串进入常量池



### 字符串常量池

> 8种基本类型的常量池都是系统协调的，`String`类型的常量池比较特殊。它的主要使用方法有两种：
>
> - 直接使用双引号声明出来的`String`对象会直接存储在常量池中。
>
> - > 如果不是用双引号声明的`String`对象，可以使用`String`提供的`intern`方法。intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中

#### 位置

从**JDK 1.7** 开始，字符串常量池不在位于方法区（永久代）中，而是被移到了虚拟堆中

#### 示例 String#intern

**注：**此示例来源于[美团技术团队文章](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)

首先需要明确的是，`String s=new String("abc");`这个语句一共创建了几个对象？

分为两种情况：

- 执行此语句前，字符串常量池中没有`"abc"`，则一共创建了两个对象
  - 第一个对象是`“abc"`字符串存储在常量池中
  - 第二对象是在Java 虚拟堆中的`String`对象
- 执行此语句前，字符串常量池种已经有`"abc"`，只创建一个对象
  - 只在队中创建一个一个`String`对象

先看一段代码：

```java
public static void main(String[] args) {
    String s = new String("1");
    s.intern();//
    String s2 = "1";
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    s3.intern();//
    String s4 = "11";
    System.out.println(s3 == s4);
}
```

执行结果为：

```text
jdk6下：false false
jdk7下：false true
```

另一段代码，更改`s.intern()`、`s3.intern()`的位置，下调一行

```java
public static void main(String[] args) {
    String s = new String("1");
    String s2 = "1";
    s.intern();//
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    String s4 = "11";
    s3.intern();//
    System.out.println(s3 == s4);
}
```

执行结果为：

```text
jdk6下：false false
jdk7下：false false
```

##### jdk6中的解释

![jdk6图](https://gitee.com/yanzixian/picBed/raw/master/img202005/20200512151249.png)

**注：**图中绿色线条代表`String`的内容指向，黑色线条代表地址指向

> 在jdk6中，常量池在Perm区（即方法区）中，Perm区与正常的java堆Heap区于是完全分开的；
>
> 如果是使用引号声明的字符串如`String s="abc"`，会直接在字符串常量池中生成，而使用`new`创建的字符串对象如`String s=new String("abc")`，会放在JAVA Heap区域；
>
> 所以拿一个 JAVA Heap 区域的对象地址和字符串常量池的对象地址进行比较肯定是不相同的，即使调用`String.intern`方法也是没有任何关系的

##### jdk7中的解释

需要注意的是，在jdk7中，字符串常量池已经从Perm区移到了正常的JAVA Heap区域

###### 第一段代码解释

![jdk7图1](https://gitee.com/yanzixian/picBed/raw/master/img202005/20200513094504.png)

> - 在第一段代码中，先看 s3和s4字符串。`String s3 = new String("1") + new String("1");`，这句代码中现在生成了最终2个对象，是字符串常量池中的“1” 和 JAVA Heap 中的 s3引用指向的对象。中间还有2个匿名的`new String("1")`我们不去讨论它们。此时s3引用对象内容是”11”，但此时常量池中是没有 “11”对象的。
> - 接下来`s3.intern();`这一句代码，是将 s3中的“11”字符串放入 String 常量池中，因为此时常量池中不存在“11”字符串，因此常规做法是跟 jdk6 图中表示的那样，在常量池中生成一个 “11” 的对象，关键点是 jdk7 中常量池不在 Perm 区域了，这块做了调整。**常量池中不需要再存储一份对象了，可以直接存储堆中的引用**。这份引用指向 s3 引用的对象。 也就是说引用地址是相同的。
> - 最后`String s4 = "11";` 这句代码中”11”是显示声明的，因此会直接去常量池中创建，创建的时候发现已经有这个对象了，此时也就是指向 s3 引用对象的一个引用。所以 s4 引用就指向和 s3 一样了。因此最后的比较 `s3 == s4` 是 true。
> - 再看 s 和 s2 对象。 `String s = new String("1");` 第一句代码，生成了2个对象。常量池中的“1” 和 JAVA Heap 中的字符串对象。`s.intern();` 这一句是 s 对象去常量池中寻找后发现 “1” 已经在常量池里了。
> - 接下来`String s2 = "1";` 这句代码是生成一个 s2的引用指向常量池中的“1”对象。 结果就是 s 和 s2 的引用地址明显不同。图中画的很清晰。

###### 第二段代码解释

![jdk7图2](https://gitee.com/yanzixian/picBed/raw/master/img202005/20200513101827.png)

> - 来看第二段代码，从上边第二幅图中观察。第一段代码和第二段代码的改变就是 `s3.intern();` 的顺序是放在`String s4 = "11";`后了。这样，首先执行`String s4 = "11";`声明 s4 的时候常量池中是不存在“11”对象的，执行完毕后，“11“对象是 s4 声明产生的新对象。然后再执行`s3.intern();`时，常量池中“11”对象已经存在了，因此 s3 和 s4 的引用是不同的。
> - 第二段代码中的 s 和 s2 代码中，`s.intern();`，这一句往后放也不会有什么影响了，因为对象池中在执行第一句代码`String s = new String("1");`的时候已经生成“1”对象了。下边的s2声明都是直接从常量池中取地址引用的。 s 和 s2 的引用地址是不会相等的

##### 小结 

从上述的例子代码可以看出 jdk7 版本对 intern 操作和常量池都做了一定的修改。主要包括2点：

- 将String常量池 从 Perm 区移动到了 Java Heap区
- `String#intern` 方法时，如果存在堆中的对象，会直接保存对象的引用，而不会重新创建对象



### 参考：

[https://blog.csdn.net/u012988901/article/details/99740694#JVM%E6%98%AF%E4%BB%80%E4%B9%88](https://blog.csdn.net/u012988901/article/details/99740694#JVM是什么)

https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html

https://www.cnblogs.com/syp172654682/p/8082625.html