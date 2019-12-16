---
title: javascript数据类型
categories:
- web前端
tags: javascript
---



### javascript数据类型

Javascript数据类型分为原始数据类型，基本数据类型和引用数据类型

#### 原始数据类型（6种）

- Undefined
- Null
- Number
- Boolean
- String
- Symbol

#### 基本数据类型（值类型，7种）

- Undefined
- Null
- Number
- Boolean
- String
- Sysbol
- <u>Object</u>

#### 引用数据类型：（3种）

- Object
- Array
- Function

#### typeof：判断数据类型，对应返回如下表：

| 数据              | 对应返回类型 |
| ----------------- | ------------ |
| String            | "string"     |
| Number            | "number"     |
| Boolean           | "boolean"    |
| Undefined         | "undefined"  |
| Null              | "null"       |
| Object            | "object"     |
| Array             | "object"     |
| function函数对象  | "function"   |
| Symbol（ES6新增） | "symbol"     |

typeof用于判断数据的类型，但对于不同的对象，它们的返回值都是object，要判断是哪个对象的实例时，用instanceof运算符