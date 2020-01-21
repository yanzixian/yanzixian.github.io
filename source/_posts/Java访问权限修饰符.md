---
title: Java访问权限修饰符
date: 2020-01-21 16:34:56
categories: java
tags:
---

Java中，可以使用访问控制符来保护对类、变量、方法和构造方法的访问。Java 支持 4 种不同的访问权限

|         作用域          | 当前类 | 同包 | 子类 | 其他 |
| :---------------------: | :----: | :--: | :--: | :--: |
|         public          |  YES   | YES  | YES  | YES  |
|        protected        |  YES   | YES  | YES  |  NO  |
| default(即什么都不写时) |  YES   | YES  |  NO  |  NO  |
|         private         |  YES   |  NO  |  NO  |  NO  |

**注：**其他是指位于其他包、且不是当前类的子类的类

