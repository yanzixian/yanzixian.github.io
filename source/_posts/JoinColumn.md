---
title: '@JoinColumn'
date: 2020-01-10 16:13:03
categories: Spring
tags: jpa
---



本文在原文的基础上，加了自己的理解和相关补充，方便自己学习和记忆，查看原文点击[此处](https://blog.csdn.net/pengjunlee/article/details/79972059)，感谢原文大佬

[toc]

### `@JoinColumn`

作用：用来指定与所操作实体或实体集合相关联的数据库表中的列字段

由于`@OneToOne`、`@OneToMany`、`@ManyToOne`、`@ManyToMany`注解只能确定不同实体之间的对应关系，不能指定对应数据库表中的关联字段，因此需要`@JoinColumn`来配合使用

使用员工、地址、部门、角色之间的关系来解释：

- 一个员工只能有一个地址，一个地址只能对应一个员工，员工与地址之间是一对一的关系
- 一个员工属于一个部门，一个部门有多个员工，部门与员工是一对多的关系，员工与部门是多对一的关系
- 一个员工有多个角色，一个角色属于多个员工，员工与角色之间是多对多的关系

##### `@OneToOne`

###### 单向

`@OneToOne` 用来表示类似于以上员工与地址之间的一对一的关系，在员工表中会有一个指向地址表主键的字段`address_id`，所以主控方（指能够主动改变关联关系的一方）一定是员工，因为，只要改变员工表的`address_id`就改变了员工与地址之间的关联关系，所以`@JoinColumn`要写在员工实体类`Employee`上，自然而然地，地址就是被控方了。



```java

@OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinColumn(name = "address_id")
private Address address;
```

我们也可以不写`@JoinColumn`，Hibernate会自动在员工表生成关联字段，字段默认的命名规则：被控方类名_被控方主键，如：`address_id`。

如果两张表是以主键关联的，比如员工表主键是`employee_id`，地址表主键是`address_id`，可以使用如下注解： 

```java

@OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@PrimaryKeyJoinColumn(name = "employee_id", referencedColumnName = "address_id")

```

主控方即关系维护端，被控方为关系被维护端，外键位于主控方。关系维护端需要负责与被维护端关系的建立，同时也要负责与被维护端关系的解除，而被维护段对这些一无所知。

在此例中，若存在员工Employee_A，对应的地址为Address_A，此时若有另一名员工Employee_B，对应的地址也为Address_A，程序是不会报错的，因为被维护端Address_A无法获知自己对应了几名员工，而员工却只知道自己只对应了一个地址。

同时，对于一组单向一对一关系，`Employee_C-->Address_C`，只删除Employee表中Employee_C这一条记录而保留Address表中的Address_C记录，程序也不会报错，原因同上；但若反过来删除Address_C而保留Employee_C，程序报错，因为Employ_C中仍然保留着指向Address_C的外键。

###### 双向

双向`@OneToOne`是在单向的基础上，使得被维护端也可以知道与自己有关的维护端的信息，只需在被维护端加上一个维护端的字段，并使用`mappedBy`属性。

此例中，在Address表（被维护端）中加入Employee（维护端）字段

```java
//mappedBy表明了关系被维护端，这样就可以通过address得到对应的employee
@OneToOne(mappedBy="address")
private Employee employee;
```

双向 `@OneToOne`中，上述示例中若有多个员工（Employee)对应同一个地址(Address)，则会报错

**总结：**

- 无论是单向还是双向，关系的保存都需要关系维护端进行操作，即上述示例中若通过保存Address来保存Employee是不会成功的，Employee的外键字段不会有值
- 所谓一对一映射，其实是关系维护端的映射关系，关系维护端只能映射一个被维护端，被维护端和几个维护端关联不能获知（单向）
- 关系维护端可以主动修改关系，而被维护端只能被动接受

##### `@OneToMany`

在分析员工与部门之间的关系时，一个员工只能属于一个部门，但是一个部门可以包含有多个员工，如果我们站在部门的角度来看，部门与员工之间就是一对多的关系，在部门实体类 `Department` 上添加如下注解： 

```java
@OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
@JoinColumn(name = "department_id")
private List<Employee> employees;
```

我们也可以不写`@JoinColumn`，`Hibernate`会自动生成一张中间表来对员工和部门进行绑定，表名默认的命名规则：一的表名_一实体类中关联多的属性名，例如，部门表名为 `tbl_department` ，部门实体中关联的员工集合属性名为 `employees` ，则生成的中间表名为：`tbl_department_employees`。

通常并不推荐让`Hibernate`自动去自动生成中间表，而是使用`@JoinTable`注解来指定中间表：  

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

###### 单向

单向 `@OneToMany`，在上述示例中表示一个部门(Department)知道拥有几个员工(Employee)，但员工却不知道自己属于哪个部门

在`One`的一方添加`@OneToMany`、`@JoinColumn`注解，`Many`的一方什么都不做

数据库表方面是在`Many`，即多的一方生成外键字段

###### 双向

双向需要使用`mappedBy`属性，并与`@ManyToOne`配合使用

在`Many`的一方使用`@ManyToOne`注解，是关系的**维护端**，是**从表**；负责外键字段的更新。在本例中即Employee，在Employee中添加外键字段，表中也生成相应的外键字段

```java
@ManyToOne
@JoinColumn(name="department_id")
private Department department;
```

在`One`的一方使用`@OneToMany`注解，是关系的**被维护端**，是**主表**；没有权利更新外键记录，在本例中即Department，在Department中使用注解`@OneToMany`和`mappedBy`属性

```java
//只要出现mappedBy属性，即为关系被维护端
@OneToMany(mappedBy="department"，cascadeType=CascadeType.all,fetch=FetchType.LAZY)
private Set<Employee> employees;
```

**注意：**

- 在`CascadeType`中的类型分别对应`EnityManager`（实体管理器）中对应的方法

- 在`xxxxToxxxx`中， 如果To后面是Many，加载方式为延迟加载， 如果To后面是One,加载方式为立即加载 

##### `@ManyToOne`

如果我们站在员工的角度来看员工与部门之间的关系时，二者之间就变成了多对一的关系，在员工实体类 `Employee` 上添加如下注解： 

```java

@ManyToOne(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
@JoinColumn(name = "department_id")
private Department department;
```

###### 单向

多个员工（Employee）属于同一个部门

在`Many`的一方，即`Employee`中添加`@ManyToOne`、`@JoinColumn`注解，`One`的一方，即`Department`中什么都不做

数据库中是在`Many`一方即`Employee`中生成外键字段

###### 双向

与`@OneToMany`配合使用，具体见`@OneToMany`双向

##### `@ManyToMany`

类似员工与角色之间的关系，一个员工可以拥有多个角色，一个角色也可以属于多个员工，员工与角色之间就是多对多的关系。通常这种多对多关系都是通过创建中间表来进行关联处理，并使用`@JoinTable`注解来指定。

###### 单向

单向`@ManyToMany`中，只需在其中一方添加`@ManyToMany`、`@JoinTable`即可，具体添加在哪一个取决于具体的需求，若需要通过员工获取其对应的多个角色，而不能通过角色获取拥有该角色的员工，只需在员工实体类中添加注解，角色实体类中什么都不做

一个员工可以拥有多个角色，在员工实体类中添加如下注解：

```java

@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinTable(name = "tbl_employee_role", joinColumns = { @JoinColumn(name = "employee_id") }, inverseJoinColumns = {
@JoinColumn(name = "role_id") })
private List<Role> roles;
```

若需要通过角色获取拥有该角色的多个员工，而不能通过员工获取该员工拥有的多个角色，需要在角色实体类中添加注解，员工实体类中什么都不做

一个角色可以属于多个员工，在角色实体类中添加如下注解：

```java
@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinTable(name = "tbl_employee_role", joinColumns = { @JoinColumn(name = "role_id") }, inverseJoinColumns = {
@JoinColumn(name = "employee_id") })
private List<Employee> employees;
```

###### 双向

双向`@ManyToMany`需要使用到`mappedBy`属性,`mappedBy`属性添加的位置有两种方式

如：

员工实体类中添加如下注解：

```java
@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinTable(name = "tbl_employee_role", joinColumns = { @JoinColumn(name = "employee_id") }, inverseJoinColumns = {
@JoinColumn(name = "role_id") })
private List<Role> roles;
```

角色实体类中添加如下注解：

```java
//mappedBy属性的值即为员工实体类中定义的private List<Role> roles;
@ManyToMany(mappedBy="roles")
private List<Employee> employees;
```

或者：

员工实体类中添加如下注解：

```java
//mappedBy属性的值即为员工实体类中定义的private List<Employee> employees;
@ManyToMany(mappedBy="empolyees")
private List<Role> roles;
```

角色实体类中添加如下注解：

```java
@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinTable(name = "tbl_employee_role", joinColumns = { @JoinColumn(name = "role_id") }, inverseJoinColumns = {
@JoinColumn(name = "employee_id") })
private List<Employee> employees;
```



### 实体类代码：

#### 单向

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

##### 地址实体类

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

#### 双向

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

##### 地址实体类

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
    
    //双向一对一，mappedBy的值即为员工实体类定义的private Address address; // 地址
    @OneToOne(mappedBy="address")
    private Employee employee;
 
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
 
    //双向 mappedBy的值即为员工实体类中定义的private Department department; // 部门
	@OneToMany(mappedBy="department")
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
 
    //双向@ManyToMany，mappedBy的值即为员工实体类中的private List<Role> roles; // 角色
	@ManyToMany(mappedBy="roles")
	private List<Employee> employees; // 拥有角色的员工
 
	// 此处省略get和set方法
 
}

```

#### 