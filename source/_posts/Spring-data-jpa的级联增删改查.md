---
title: Spring-data-jpa的级联增删改查
date: 2020-03-13 14:14:23
categories: springboot
tags:
---

### 概述

此文是为了研究jpa级联表之间的增删改查，对这一方面内容进行一个总结

为了方便描述，使用的数据结构比较简单，书店和书，双向一对多的关系，实体如下：

#### 默认情况下：（即不设置cascade属性，不显式使用CascadeType)

Book实体

```java
@Data
@Entity
@Table
public class Book implements Serializable {

    /**
     * 编号
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    /**
     * 书名
     */
    @Column(name="name")
    private String name;

    @ManyToOne
    @JoinColumn(name="bookstore_id")
    private BookStore bookStore;
}
```

BookStore实体

```java
@Data
@Entity
@Table
public class BookStore implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    @Column(name="name")
    private String name;

    @OneToMany(mappedBy = "bookStore")
    private List<Book> bookList;

}

```

初始数据库表：
book表

![image-20200313162043590](https://gitee.com/yanzixian/picBed/raw/master/img202003/book表.png)

book_store表

![image-20200313162158955](https://gitee.com/yanzixian/picBed/raw/master/img202003/book_store表.png)

##### save操作

###### 只对book_store进行保存

测试代码

```java
@Test
public void test1(){

   Book b=new Book();
   b.setName("book1");

   BookStore bs=new BookStore();
   bs.setName("store1");

   bookStoreRepo.save(bs);
}
```

保存后数据库如下：

book表：

![image-20200313163415366](https://gitee.com/yanzixian/picBed/raw/master/img202003/book1.png)

book_store表：
![image-20200313163453556](https://gitee.com/yanzixian/picBed/raw/master/img202003/bookstore1.png)

可以看到只有book_store中有数据（id为4是因为之前表里有数据，然后手动进行了删除，所以id从4开始，不要在意这些细节），book表仍为空，说明保存关系被维护端并不会保存对应的关系维护端

###### 只对book进行保存

测试代码

```java
public void test2(){
   Book b=new Book();
   b.setName("book2");

   BookStore bs=new BookStore();
   bs.setName("store2");

   b.setBookStore(bs);
   bookRepo.save(b);
}
```

出现错误

`TransientPropertyValueException: object references an unsaved transient instance - save the transient instance before flushing`

错误原因为对b进行持久化（即保存）之前为对bs进行持久化

###### 先保存bookstore在保存book

测试代码

```java
public void test3(){
		Book b=new Book();
		b.setName("book3");

		BookStore bs=new BookStore();
		bs.setName("store3");

		b.setBookStore(bs);

		bookStoreRepo.save(bs);
		bookRepo.save(b);
	}
```

操作成功，查看数据库

book表

![image-20200316092924213](https://gitee.com/yanzixian/picBed/raw/master/img202003/book3.png)

bookstore表

![image-20200316093031566](https://gitee.com/yanzixian/picBed/raw/master/img202003/bookstore3.png)

可以看到，数据，外键均保存成功

###### 先保存book在保存bookstore

测试代码

```java
public void test4(){
   Book b=new Book();
   b.setName("book4");

   BookStore bs=new BookStore();
   bs.setName("store4");

   b.setBookStore(bs);

   bookRepo.save(b);
   bookStoreRepo.save(bs);

}
```

操作失败，错误原因与只保存book相同

###### 总结：

默认情况下，保存关系维护端由于外键的存在，必须后于被维护端进行保存，否则报错；

保存需要注意顺序，但均需进行保存操作；不会出现只保存一端，另一端也得到保存的情况，即保存没有级联

##### delete操作

执行操作前数据库为：

book表：

![image-20200316104553527](https://gitee.com/yanzixian/picBed/raw/master/img202003/book4.png)

bookstore表

![image-20200316103511017](https://gitee.com/yanzixian/picBed/raw/master/img202003/bookstore.png)

###### 对bookstore表进行删除

测试代码

```java
public void test5(){

      Integer id=2;
      BookStore bs1= bookStoreRepo.findById(id.longValue()).get();
      id=3;
      BookStore bs2= bookStoreRepo.findById(id.longValue()).get();
      id=4;
      BookStore bs3= bookStoreRepo.findById(id.longValue()).get();

      //操作1
      bookStoreRepo.delete(bs1);
      //操作2
//    bookStoreRepo.delete(bs2);
      //操纵3
//    bookStoreRepo.delete(bs3);

   }
```

测试代码中，本别对id为2，3，4的bookstore记录进行删除，这几个记录分别关联book表中的2个，1个，0个记录

其中，操作1，2均执行失败，原因为book表中有指向它们的外键，存在外键约束；操作3执行成功，原因为book表中不存在指向它们的外键，没有外键约束，执行后book表不变，bookstore表如下：

![image-20200316105134495](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316105134495.png)

###### 对book进行删除

测试代码

```java
public void test6(){

   Integer id=1;
   Book b1= bookRepo.findById(id.longValue()).get();
   id=2;
   Book b2=bookRepo.findById(id.longValue()).get();
   id=3;
   Book b3=bookRepo.findById(id.longValue()).get();
   id=5;
   Book b5=bookRepo.findById(id.longValue()).get();
   id=6;
   Book b6=bookRepo.findById(id.longValue()).get();

   bookRepo.delete(b1);
   bookRepo.delete(b2);
   bookRepo.delete(b3);
   bookRepo.delete(b5);
   bookRepo.delete(b6);


}
```

对book表中id为1，2，3，5，6的记录进行删除，操作成功；操作后bookstore表不变，book表如下：

![image-20200316105924109](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316105924109.png)

可以看到记录被成功删除

###### 总结

对关系被维护端（bookstore）进行删除时，若关系维护端（book）存在指向要删除记录的外键，由于外键约束，操作会失败；

对关系维护端（book）删除时，则不需要考虑外键的影响

对关系维护端的删除（若删除成功）不会导致关系被维护端记录的删除，反过来也不影响，即删除没有级联

##### update操作

数据库表采用上述delete操作之前表的初始状态

###### 对Book对象进行更新

测试代码

```java
@Test
public void test15(){

   Integer id=1;
   Book b1= bookRepo.findById(id.longValue()).get();
   id=3;
   Book b2= bookRepo.findById(id.longValue()).get();
   id=5;
   Book b3=bookRepo.findById(id.longValue()).get();

   b1.setName("book1_new");
   b1.getBookStore().setName("store1_new");

   b2.setName("book2_new");
   b2.getBookStore().setName("store2_new");

   b3.setName("book3_new");

   bookRepo.save(b1);
   bookRepo.save(b2);
   bookRepo.save(b3);
}
```

更新完成后各表为：

book表

![image-20200317135045425](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317135045425.png)

bookstore表

![image-20200317135152263](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317135152263.png)

###### 对BookStore对象进行更新

测试代码

```java
@Test
public void test16(){

   Integer id=1;
   BookStore bs1= bookStoreRepo.findById(id.longValue()).get();
   id=2;
   BookStore bs2= bookStoreRepo.findById(id.longValue()).get();
   id=3;
   BookStore bs3= bookStoreRepo.findById(id.longValue()).get();


   bs1.setName("store1_new");
   bs1.getBookList().get(0).setName("book1_new");

   bs2.setName("store2_new");
   bs2.getBookList().get(0).setName("book3_new");

   bs3.setName("store3_new");
   bs3.getBookList().get(0).setName("book4_new");

   bookStoreRepo.save(bs1);

   bookStoreRepo.save(bs2);

   bookStoreRepo.save(bs3);

}
```

更新完成各表为：
book表

![image-20200317135601789](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317135601789.png)

bookstore表

![image-20200317135719870](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317135719870.png)

###### **总结：**

默认情况下的更新操作只限定于当前实体中，即一个实体的更新会导致另一实体的数据的改变

为了保持数据的一致，需要涉及数据改变的实体均进行一次更新操作，即更新没有级联

#### 级联保存操作（`CascadeType.PERSIST`）

##### 只在关系维护端（book）增加`CascadeType.PERSIST`

在Book实体上设置cascade属性，BookStore实体保持不变：

Book实体

```java
@Data
@Entity
@Table
public class Book implements Serializable {

    /**
     * 编号
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    /**
     * 书名
     */
    @Column(name="name")
    private String name;
    
	//添加cascade属性
    @ManyToOne(cascade = CascadeType.PERSIST)
    @JoinColumn(name="bookstore_id")
    private BookStore bookStore;
}
```



**操作前数据库表数据为空**

###### **只对Book对象进行保存**

测试代码

```java
public void test7(){

   Book b=new Book();
   b.setName("book1");

   BookStore bs=new BookStore();
   bs.setName("store1");

   b.setBookStore(bs);

   bookRepo.save(b);
}
```

操作成功，操作后数据库如下：

book表

![image-20200316112726938](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316112726938.png)

bookstore表

![image-20200316112759295](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316112759295.png)

可以看到，两个表中数据均保存成功，外键也正确设置

###### **只对BookStore对象进行保存**

测试代码

```java
public void test8(){

   Book b=new Book();
   b.setName("book2");

   BookStore bs=new BookStore();
   bs.setName("store2");

   b.setBookStore(bs);
   
   bookStoreRepo.save(bs);
}
```

操作成功，数据库结果如下：

book表

![image-20200316134215353](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316134215353.png)

bookstore表

![image-20200316134313700](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316134313700.png)

只有bookstore表中增加了新的数据，book表没有改变

###### **总结：**

在关系维护端Book实体上添加CascadeType.PERSIST属性，保存Book实体的同时会将与其关联的实体（BookStore）一起进行保存

##### 只在关系被维护端（bookstore）增加`CascadeType.PERSIST`

在BookStore实体上设置CascadeType.PERSIST，Book实体保持不变

BookStore实体

```java
@Data
@Entity
@Table
public class BookStore implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    @Column(name="name")
    private String name;
    
	//设置cascade属性
    @OneToMany(mappedBy = "bookStore",cascade = CascadeType.PERSIST)
    private List<Book> bookList;

}
```

**操作前数据库表数据为空**

###### 只对BookStore对象进行保存

测试代码

```java
public void test9(){

   Book b=new Book();
   b.setName("book1");

   Book b1=new Book();
   b1.setName("book2");

   BookStore bs=new BookStore();
   bs.setName("store1");
    //不加下面两行会导致book表中外键为空
   b.setBookStore(bs);
   b1.setBookStore(bs);

   bs.setBookList(Arrays.asList(b,b1));

   bookStoreRepo.save(bs);
}
```

操作成功，数据库保存成功，操作完成后数据库如下：

book表

![image-20200316142914670](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316142914670.png)

bookstore表

![image-20200316142952323](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316142952323.png)

可以看到数据成功保存，外键也成功添加

**注意：**

在实际操作过程中，因为漏加下面两句，虽然数据库数据可以成功保存，但book表中外键列为空，原因个人认为是BookStore为被维护端，无法负责外键的更新

```java
b.setBookStore(bs);
b1.setBookStore(bs);
```

###### 只对Book对象进行保存

测试代码

```java
public void test10(){

   Book b=new Book();
   b.setName("book3");

   BookStore bs=new BookStore();
   bs.setName("store2");

   b.setBookStore(bs);

   bookRepo.save(b);
}
```

程序报错，报错原因与"默认情况下的只对Book对象进行保存"报错原因相同

##### 在维护端和被维护端均设置`CascadeType.PERSIST`

Book实体

```java
@Data
@Entity
@Table
public class Book implements Serializable {

    /**
     * 编号
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    /**
     * 书名
     */
    @Column(name="name")
    private String name;
    
	//添加cascade属性
    @ManyToOne(cascade = CascadeType.PERSIST)
    @JoinColumn(name="bookstore_id")
    private BookStore bookStore;
}
```

BookStore实体

```java
@Data
@Entity
@Table
public class BookStore implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    @Column(name="name")
    private String name;
    
	//添加cascade属性
    @OneToMany(mappedBy = "bookStore",cascade = CascadeType.PERSIST)
    private List<Book> bookList;

}
```

**操作前数据库表数据为空**

###### 只对Book对象进行保存

测试代码

```java
public void test7(){

   Book b=new Book();
   b.setName("book1");

   BookStore bs=new BookStore();
   bs.setName("store1");

   b.setBookStore(bs);

   bookRepo.save(b);
}
```

操作成功，数据保存成功

book表

![image-20200316145216667](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316145216667.png)

bookstore表

![image-20200316145244420](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316145244420.png)

###### 只对BookStore对象进行保存

测试代码

```java
public void test9(){

   Book b=new Book();
   b.setName("book2");

   Book b1=new Book();
   b1.setName("book3");

   BookStore bs=new BookStore();
   bs.setName("store2");
    //不加下面两行会导致book表中外键为空
   b.setBookStore(bs);
   b1.setBookStore(bs);

   bs.setBookList(Arrays.asList(b,b1));

   bookStoreRepo.save(bs);
}
```

操作成功，数据库表如下：

book表

![image-20200316145642382](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316145642382.png)

bookstore表

![image-20200316145722120](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316145722120.png)

##### **总结：**

cascade属性的设置即赋予了当前实体操作与之关联的另一实体的权限

属性值为`CascadeType.PERSIST`时，即赋予当前实体保存另一实体的权限，即保存当前实体时，也会级联保存与之关联的实体

在本例中，在Book实体中设置bookStore的cascade属性为`CacadeType.PERSIST`，即赋予Book保存BookStore实体的权限；在BookStore实体中设置bookList的cascade属性为`CacadeType.PERSIST`，即赋予BookStore保存Book实体的权限；但同时也要注意维护端和被维护端的不同之处，外键的更新只能由维护端来负责

#### 级联删除操作（`CascadeType.REMOVE`）

##### 只在关系维护端（book）设置`CascadeType.REMOVE`

在Book实体中设置cascade

```java
@Data
@Entity
@Table
public class Book implements Serializable {

    /**
     * 编号
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    /**
     * 书名
     */
    @Column(name="name")
    private String name;
	//设置cascade属性
    @ManyToOne(cascade = CascadeType.REMOVE)
    @JoinColumn(name="bookstore_id")
    private BookStore bookStore;
```

###### 只对book进行删除

删除前各表为：

book表

![image-20200316152944391](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316152944391.png)

bookstore表

![image-20200316153015585](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316153015585.png)

测试代码

```java
public void test11(){

		Integer id=1;
		Book b1= bookRepo.findById(id.longValue()).get();
		id=2;
		Book b2= bookRepo.findById(id.longValue()).get();
		id=3;
		Book b3=bookRepo.findById(id.longValue()).get();

		List<Book> list=new ArrayList<>();
		list.add(b1);
		list.add(b2);

		bookRepo.deleteAll(list);
		bookRepo.delete(b3);
	}
```

操作成功，数据库结果如下：

book表

![image-20200316160945428](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316160945428.png)

bookstore表

![image-20200316161012313](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316161012313.png)

两个表均为空，数据全部删除，实现级联删除

**注意：**

设置`CascadeType.REMOVE`，实体删除时会删除与之相关联的实体

如上book表中，id为1，2的记录均有指向bookstore表中id=1的记录的外键；当单独删除book1时，理论上会导致与之关联的store1删除，但又由于book2有指向store1的外键，故这样会导致book2外键指向为空，会报错；单独删除book2同理；

所以单独删除是不会成功的，故上述测试代码使用了一个List，使用deleteAll进行全部删除；

删除过程为先更新book中所有相关记录的外键为空值，再删除一条book记录，再删除bookstore记录；有多个book记录时，先删除一条book记录，再删除bookstore的记录，再删除剩余的book记录

###### 只对bookstore进行删除

操作前数据库表为：

book表

![image-20200316162825881](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316162825881.png)

bookstore表

![image-20200316162848602](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316162848602.png)

测试代码

```java
public void test12(){

      Integer id=1;
      BookStore bs1= bookStoreRepo.findById(id.longValue()).get();
      id=2;
      BookStore bs2= bookStoreRepo.findById(id.longValue()).get();


//    操作1
      bookStoreRepo.delete(bs1);
      //操作2
//    bookStoreRepo.delete(bs2);


   }
```

操作1删除store1记录失败，因为book表中有指向它的外键；表里数据均不变

操作2删除store2成功，没有键约束，book表不变；bookstore表如下：

![image-20200316163218388](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316163218388.png)

##### 只在关系被维护端（bookstore）设置`CascadeType.REMOVE`

在BookStore实体中设置cascade

```java
@Entity
@Table
public class BookStore implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    @Column(name="name")
    private String name;

    @OneToMany(mappedBy = "bookStore",cascade = CascadeType.REMOVE)
    private List<Book> bookList;

}
```

###### 只对book进行删除

删除前数据库表为：
book表

![image-20200316163901618](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316163901618.png)

bookstore表

![image-20200316163926101](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316163926101.png)

测试代码

```java
@Test
public void test13(){

   Integer id=1;
   Book b1= bookRepo.findById(id.longValue()).get();
   id=3;
   Book b2= bookRepo.findById(id.longValue()).get();
   id=5;
   Book b3=bookRepo.findById(id.longValue()).get();

   bookRepo.delete(b1);
   bookRepo.delete(b2);
   bookRepo.delete(b3);
}
```

对book表中id为1，3，5的记录进行删除，操作成功，操作完成数据库表为：

book表

![image-20200317092519784](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317092519784.png)

bookstore未改变

###### 只对bookstore进行删除

删除前数据库表与上述初始相同

book表

![image-20200316163901618](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316163901618.png)

bookstore表

![image-20200316163926101](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316163926101.png)

测试代码

```java
@Test
public void test14(){

   Integer id=1;
   BookStore bs1= bookStoreRepo.findById(id.longValue()).get();
   id=2;
   BookStore bs2= bookStoreRepo.findById(id.longValue()).get();
   id=3;
   BookStore bs3= bookStoreRepo.findById(id.longValue()).get();

   //操作1
   bookStoreRepo.delete(bs1);
   //操作2
   bookStoreRepo.delete(bs2);
   //操作3
   bookStoreRepo.delete(bs3);

}
```

对bookstore表中id为1，2，3的记录进行删除，操作成功， 操作完成后数据库表为：

book表

![image-20200317093439406](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317093439406.png)

bookstore表

![image-20200317093507329](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317093507329.png)

可以看到bookstore中的记录删除成功，同时book表中与之相关联的记录也被删除，实现了级联删除

##### **总结：**

cascade属性值为`CascadeType.REMOVE`时，赋予了当前实体删除另一实体中记录的权限，即删除当前实体记录时，也会删除级联实体中与之相关联（外键映射）的记录

没有设置`CascadeType.REMOVE`的实体进行删除时与默认情况相同

关系维护端和关系被维护端进行级联删除时情况稍有不同，关系维护端进行级联删除时要注意将拥有相同外键映射的记录一起删除，否则会因为外键约束的原因而导致操作失败，但这种删除操作实际中一般使用场景不多；

#### 级联更新操作（`CascadeType.MERGE`）

##### 在关系维护端(book)设置cascade=`CascadeType.MERGE`

在Book实体中设置cascade

```java
@Data
@Entity
@Table
public class Book implements Serializable {

    /**
     * 编号
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    /**
     * 书名
     */
    @Column(name="name")
    private String name;
	//设置cascade
    @ManyToOne(cascade = CascadeType.MERGE)
    @JoinColumn(name="bookstore_id")
    private BookStore bookStore;
}
```

###### 对Book对象进行更新

更新前表为

book表

![image-20200316163901618](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316163901618.png)

bookstore表

![image-20200316163926101](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200316163926101.png)

测试代码

```java
@Test
public void test15(){

   Integer id=1;
   Book b1= bookRepo.findById(id.longValue()).get();
   id=3;
   Book b2= bookRepo.findById(id.longValue()).get();
   id=5;
   Book b3=bookRepo.findById(id.longValue()).get();

   b1.setName("book1_new");
   b1.getBookStore().setName("store1_new");

   b2.setName("book2_new");
   b2.getBookStore().setName("store2_new");

   b3.setName("book3_new");

   bookRepo.save(b1);
   bookRepo.save(b2);
   bookRepo.save(b3);
}
```

更新成功，更新后各表为：

book表

![image-20200317102542745](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317102542745.png)

bookstore表

![image-20200317102720012](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317102720012.png)

可以看到，更改了Book对象中BookStore属性的name属性，保存对象时不但book表得到更新，bookstore也相应地更新，实现了级联更新

###### 对BookStore对象进行更新

更新前表恢复为原状

测试代码

```java
@Test
public void test16(){

   Integer id=1;
   BookStore bs1= bookStoreRepo.findById(id.longValue()).get();
   id=2;
   BookStore bs2= bookStoreRepo.findById(id.longValue()).get();
   id=3;
   BookStore bs3= bookStoreRepo.findById(id.longValue()).get();


   bs1.setName("store1_new");
   bs1.getBookList().get(0).setName("book1_new");

   bs2.setName("store2_new");
   bs2.getBookList().get(0).setName("book3_new");

   bs3.setName("store3_new");
   bs3.getBookList().get(0).setName("book4_new");

   bookStoreRepo.save(bs1);

   bookStoreRepo.save(bs2);

   bookStoreRepo.save(bs3);

}
```

操作成功，更新后各表为：

book表

![image-20200317104912393](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317104912393.png)

bookstore表

![image-20200317105048380](https://gitee.com/yanzixian/picBed/raw/master/img202003/image-20200317105048380.png)

可以看到更改BookStore对象中的name属性、以及其中的Book属性的name时，只有bookstore表得到了更新

##### 在关系被维护端(BookStore)设置cascade=CascadeType.MERGE

在BookStore实体中设置cascade

```java
@Data
@Entity
@Table
public class BookStore implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="id")
    private Long id;

    @Column(name="name")
    private String name;
	//设置cascade
    @OneToMany(mappedBy = "bookStore",fetch = FetchType.EAGER,cascade = CascadeType.MERGE)
    private List<Book> bookList;

}
```

还是采用上述的数据库表，和测试代码test15、test16

可以得到结论正好与上述情况相反，test15没有实现级联更新，test16实现了级联更新

##### **总结：**

cascade属性值为`CascadeType.MERGE`时，赋予了当前实体更新另一实体的权限，即更改当前实体对象中代表另一实体对象的属性时（如更改Book对象中的BookStore属性），可以时另一实体的数据库表也得到更新，即级联更新，无需再单独进行更新

若需要两个实体对象可以互相更新，只需两个实体类中都设置`CascadeType.MERGE`即可

---

（ps  还差个REFRESH，写不下去了，有时间再看吧）

![](https://gitee.com/yanzixian/picBed/raw/master/img202003/20200317141034.png)

2020/3/17

---

