---
title: HQL与SQL
date: 2020-02-15 18:38:51
categories: hibernate
tags: sql
---

### HQL(Hibernate Query Language)

#### 概述

- 完全面对对象的查询语句，查询功能强大，具备多态、关联等特性

- Hibernate官方推荐

- HQL独立于数据库，hibernate根据方言把hql生成对应的sql语句，不能直接与数据库进行交互，属于中间层的语言

  ![image-20200309152525583](https://gitee.com/yanzixian/picBed/raw/master/img202003/hql工作流程示意.png)

- 支持命名参数

- 对查询条件进行了面对对象封装，格式

  ```text
  select 类对象（类对象属性） from 类名 类对象 where 对象的属性
  ```

- 区分大小写，关键字不区分大小写

- 从下标0开始计算位置（hibernate5之后不支持）
- HQL语言不能直接进行`insert`操作，`select`，`delete`，`update`支持

#### 返回类型

##### 单个对象

```java
public static void test1(){
   Session session = SessionFactoryUtils.openSession();
   Transaction transaction = session.beginTransaction();
    //查询语句，根据id查询，只会查到一条记录
        String hql="from Student where sid=1";
    	//创建一个查询
        Query query = session.createQuery(hql);
    /**使用list()方式
    List<Student> list=(Studnet)query.list().get(0);
    query.list()返回的是一个集合，此时集合中只有一个对象，通过下标0取出该对象，需要强转成Student对象，因为query.list().get(0)拿到的是Object类型的对象，向下转型成子类Student对象。

这种方式存在一个问题，若id=0时，查询不到对象，集合为空，使用get(0)会抛出下标越界异常
推荐使用另外一种方式，通过uniqueResult()方法，该方法返回一个Object对象，如果对象不存在则返回null，不会抛出异常
    */
        Object obj = query.getSingleResult();
    
        System.out.println(obj.getClass().getName());
        System.out.println(obj);
        transaction.commit();
        SessionFactoryUtils.closeSession();
    }
```

##### 多个对象

```java
/**
     * hql语句结果集处理情况1
     * 返回多个对象  注意数据库字段不能为null
     */
    public static void test2(){
        Session session = SessionFactoryUtils.openSession();
        Transaction transaction = session.beginTransaction();
        String hql="from Student";
        Query query = session.createQuery(hql);
        List list=query.list();
        for (Object o : list) {
            System.out.println(o);
        }
        transaction.commit();
        SessionFactoryUtils.closeSession();
}
```

##### 返回字符串

```java
public static void test3(){
        Session session = SessionFactoryUtils.openSession();
        Transaction transaction = session.beginTransaction();
    //查询一个字段，用String接收对象
        String hql="select sname from Student where sid=1";
        Query query = session.createQuery(hql);
        Object obj=query.getSingleResult();
    //使用list()方式
    //List<String> list = query.list();
        System.out.println(obj);
        transaction.commit();
        SessionFactoryUtils.closeSession();
    }

```

##### 返回数组

```java
public static void test4(){
        Session session = SessionFactoryUtils.openSession();
        Transaction transaction = session.beginTransaction();
        //查询多个字段，用数组接收
        String hql="select sid,sname,version from Student where sid=1";
        Query query = session.createQuery(hql);
        Object obj=query.getSingleResult();
    /**使用list()方式
    List<Object[]> list = query.list();
        for (Object[] b : list) {
            System.out.println(Arrays.toString(b));
        }
        */
        System.out.println(Arrays.toString((Object[])obj));
        transaction.commit();
        SessionFactoryUtils.closeSession();
    }

```

##### 查询多列，构造对象返回（需要对应的构造函数）

```java
public static void test6(){
        Session session = SessionFactoryUtils.openSession();
        Transaction transaction = session.beginTransaction();
    //构造一个包含查询字段的对象,前提是有对应的构造函数
        String hql="select new Student (sid,sname)from Student where sid=1";
        Query query = session.createQuery(hql);
        Object obj=query.getSingleResult();
        System.out.println(obj.getClass().getName());
    /**使用list()的方法
       List<Book> list = query.list();
        for (Book b : list) {
            System.out.println(b);
        }
    */
        System.out.println(obj);
        transaction.commit();
        SessionFactoryUtils.closeSession();
    }

```

##### 返回Map

```java
public static void test5(){
        Session session = SessionFactoryUtils.openSession();
        Transaction transaction = session.beginTransaction();
    //该查询返回了一个Map的对象，内容是别名与被选择的值组成的名-值映射
    //使用关键字as给“被选择了的表达式”指派别名
        String hql="select new map(sid as sid,sname as sname,version as version) from Student where sid=1";
        Query query = session.createQuery(hql);
        Object obj=query.getSingleResult();
        System.out.println(obj);
        transaction.commit();
        SessionFactoryUtils.closeSession();
    }

```

上述代码来自[原文](https://blog.csdn.net/qq_41594146/article/details/84573822)，感谢大佬

![](https://gitee.com/yanzixian/picBed/raw/master/img202003/感谢大佬.png)

---

#### ：命名参数

hql查询语句中常用`？`占位符，如

```hql
from User user where user.name=? and user.age=? and ...;
```

在设置参数的时候，就需要使用如下方式设置参数：

```java
query.setString(0,wang);
query.setInt(1,11);
...
    ...
```

这样的方式需要注意参数的顺序，在参数多的情况下容易出错，可读性不高

因此可以使用自定义参数名称的方式，如

```java
from User user where user.name= :name and user.age= :age and ...;
```

按如下方式设置参数

```
query.setString("name",wang);
query.setInt("age",11);
...
...
```

这样即使顺序写反，也不不会出现对应错误的问题，可读性高

#### 聚合函数

HQL聚合函数有:

count()：统计记录条数

sum()：求和

max()：求最大值

min()：求最小值

avg()：求平均值

##### count操作：统计符合条件记录数（其余操作类似）：

```java
public void test() {
		//count  统计符合条件的记录数
		String hql = "select count(*) from Book";
		Object result = session.createQuery(hql).getSingleResult();//查单行单列
		System.out.println(result);
	}
```


