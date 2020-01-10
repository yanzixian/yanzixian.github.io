---
title: '@JoinColumn'
date: 2020-01-10 16:13:03
categories: Spring
tags: jpa
---

本文在原文的基础上，加了自己的理解和相关补充，方便自己学习和记忆，查看原文点击[此处](https://blog.csdn.net/pengjunlee/article/details/79972059)，感谢原文大佬

#### `@JoinColumn`

作用：用来指定与所操作实体或实体集合相关联的数据库表中的列字段

由于`@OneToOne`、`@OneToMany`、`@ManyToOne`、`@ManyToMany`注解只能确定不同实体之间的对应关系，不能指定对应数据库表中的关联字段，因此需要`@JoinColumn`来配合使用

使用员工、地址、部门、角色之间的关系来解释：

- 一个员工只能有一个地址，一个地址只能对应一个员工，员工与地址之间是一对一的关系
- 一个员工属于一个部门，一个部门有多个员工，部门与员工是一对多的关系，员工与部门是多对一的关系
- 一个员工有多个角色，一个角色属于多个员工，员工与角色之间是多对多的关系

##### `@OneToOne`

@OneToOne 用来表示类似于以上员工与地址之间的一对一的关系，在员工表中会有一个指向地址表主键的字段address_id，所以主控方（指能够主动改变关联关系的一方）一定是员工，因为，只要改变员工表的address_id就改变了员工与地址之间的关联关系，所以@JoinColumn要写在员工实体类Employee上，自然而然地，地址就是被控方了。

```java

@OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinColumn(name = "address_id")
private Address address;
```

我们也可以不写@JoinColumn，Hibernate会自动在员工表生成关联字段，字段默认的命名规则：被控方类名_被控方主键，如：address_id。

如果两张表是以主键关联的，比如员工表主键是employee_id，地址表主键是address_id，可以使用如下注解： 

```java

@OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@PrimaryKeyJoinColumn(name = "employee_id", referencedColumnName = "address_id")

```

##### `@OneToMany`

在分析员工与部门之间的关系时，一个员工只能属于一个部门，但是一个部门可以包含有多个员工，如果我们站在部门的角度来看，部门与员工之间就是一对多的关系，在部门实体类 Department 上添加如下注解： 

```java
@OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
@JoinColumn(name = "department_id")
private List<Employee> employees;
```

我们也可以不写@JoinColumn，Hibernate会自动生成一张中间表来对员工和部门进行绑定，表名默认的命名规则：一的表名_一实体类中关联多的属性名，例如，部门表名为 tbl_department ，部门实体中关联的员工集合属性名为 employees ，则生成的中间表名为：tbl_department_employees。

通常并不推荐让Hibernate自动去自动生成中间表，而是使用@JoinTable注解来指定中间表：  

```java
@OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
/**
* joinColumns 用来指定中间表中关联自己ID的字段 
* inverseJoinColumns 用来指定中间表中关联对方ID的字段
*/
@JoinTable(name = "tbl_employee_department", joinColumns = {
@JoinColumn(name = "department_id") }, inverseJoinColumns = { @JoinColumn(name = "employee_id") })
private List<Employee> employees;
```

##### `@ManyToOne`

如果我们站在员工的角度来看员工与部门之间的关系时，二者之间就变成了多对一的关系，在员工实体类 Employee 上添加如下注解： 

```java

@ManyToOne(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
@JoinColumn(name = "department_id")
private Department department;
```

`@ManyToMany`

类似员工与角色之间的关系，一个员工可以拥有多个角色，一个角色也可以属于多个员工，员工与角色之间就是多对多的关系。通常这种多对多关系都是通过创建中间表来进行关联处理，并使用@JoinTable注解来指定。

一个员工可以拥有多个角色，在员工实体类中添加如下注解：

```java

@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinTable(name = "tbl_employee_role", joinColumns = { @JoinColumn(name = "employee_id") }, inverseJoinColumns = {
@JoinColumn(name = "role_id") })
private List<Role> roles;

```

一个角色可以属于多个员工，在角色实体类中添加如下注解：

```java
@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinTable(name = "tbl_employee_role", joinColumns = { @JoinColumn(name = "role_id") }, inverseJoinColumns = {
@JoinColumn(name = "employee_id") })
private List<Employee> employees;
```

#### 实体类代码：

##### 员工实体类

```java
@Entity
@Table(name = "tbl_employee") // 指定关联的数据库的表名
public class Employee {
 
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id; // 主键ID
 
	private String name; // 姓名
 
	@DateTimeFormat(pattern = "yyyy-MM-dd")
	private LocalDate birthday; // 生日
 
	@OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
	@JoinColumn(name = "address_id")
	// @PrimaryKeyJoinColumn(name = "employee_id", referencedColumnName = "address_id")
	private Address address; // 地址
 
	@ManyToOne(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
	@JoinColumn(name = "department_id")
	private Department department; // 部门
 
	@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
	@JoinTable(name = "tbl_employee_role", joinColumns = { @JoinColumn(name = "employee_id") }, inverseJoinColumns = {
			@JoinColumn(name = "role_id") })
	private List<Role> roles; // 角色
 
	// 此处省略get和set方法
}

```

##### 地址实体类：

```java
@Entity
@Table(name = "tbl_address")
public class Address {
 
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id; // 主键ID
 
	private String province; // 省
 
	private String city; // 市
 
	private String district; // 区
 
	private String address; // 详细地址
 
	// 此处省略get和set方法
 
}

```

##### 部门实体类

```java
@Entity
@Table(name = "tbl_department")
public class Department {
 
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id; // 主键ID
 
	private String name; // 部门名称
 
	@OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
	@JoinColumn(name = "department_id")
	/**
	 * joinColumns 用来指定中间表中关联自己ID的字段 inverseJoinColumns 用来指定中间表中关联对方ID的字段
	 */
	// @JoinTable(name = "tbl_employee_department", joinColumns = {
	// 		@JoinColumn(name = "department_id") }, inverseJoinColumns = { @JoinColumn(name = "employee_id") })
	private List<Employee> employees; // 部门员工
 
	// 此处省略get和set方法
}
```

##### 角色实体类

```java
@Entity
@Table(name = "tbl_role")
public class Role {
 
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id; // 主键ID
 
	private String name; // 角色名称
 
	@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
	@JoinTable(name = "tbl_employee_role", joinColumns = { @JoinColumn(name = "role_id") }, inverseJoinColumns = {
			@JoinColumn(name = "employee_id") })
	private List<Employee> employees; // 拥有角色的员工
 
	// 此处省略get和set方法
 
}

```

