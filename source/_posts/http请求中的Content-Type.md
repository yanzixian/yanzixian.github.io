---
title: http请求中的Content-Type
date: 2020-04-16 09:27:20
categories: web前端
tags:
---

#### 前言

写`vue`项目用`axios`发请求的时候一直对请求中的`Content-Type`理解得不清楚，用的时候很混乱（虽然也没出啥错，但谁知道以后呢），这里理清一下

![](https://gitee.com/yanzixian/picBed/raw/master/img202004/20200416094547.png)

#### Content-Type

##### 概述

MediaType，即Internet Media Type，互联网媒体类型，也叫MIME类型，在http消息头中（请求和响应消息中都有），使用`Content-Type`来表示具体请求中的媒体类型

常见的媒体格式类型如下：

- text/html：HTML格式
- text/plain：纯文本格式
- text/xml：XML格式
- image/gif：gif图片格式
- image/jpeg：jpg图片格式
- image/png：png图片格式

以application开头的媒体格式类型：

- application/xhtml+xml：XHTML格式
- application/xml：XML数据格式
- application/atom+xml：Atom XML聚合格式
- application/json：JSON数据格式
- application/pdf：pdf格式
- application/msword：word文档格式
- application/octet-stream：二进制流数据（常见的文件下载）
- application/x-www-form-urlencoded： `<form encType="">`中默认的encType，不设置`encType`属性，即为此类型；form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）

另上传文件使用的媒体格式：

- multipart/form-data：表单中进行文件上传时，使用此格式

##### 请求头中的`Content-Type`

作为请求消息头中的一个参数，标识请求消息数据的格式，如`Content-Type: text/html;charset:utf-8;`

###### get请求

`Content-Type`用于标识请求体中的数据格式，而`get`请求将请求参数拼接在url后面，没有请求体数据，故一般不需要设置`Content-Type`

###### post请求

常见数据提交格式：

- application/x-www-form-urlencoded

  最常见的post提交数据的方式，浏览器的原生`<form>`表单，如果不设置`encType`属性，将以此方式提交数据：编码为key/value格式，key与value均会被urlencoded

- application/form-data

  另一个常见的post数据提交的方式，需要上传文件时，必须使用此格式；此格式中会有boundary属性，用于进行隔离

  `Content-Type: multipart/form-data; boundary=--------------------------262130595218932429345449`

- application/json

  可以方便的提交复杂的结构化数据，告诉服务端消息主体时序列化后的JSON字符串

**注：**此处[原文](https://www.cnblogs.com/xingzc/p/6277550.html)

postman中form-data、x-www-form-urlencoded、raw、binary的区别

- form-data：

  就是http请求中的`multipart/form-data`,它会将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件。当上传的字段是文件时，会有Content-Type来说明文件类型；content-disposition，用来说明字段的一些信息；

  由于有boundary隔离，所以multipart/form-data既可以上传文件，也可以上传键值对，它采用了键值对的方式，所以可以上传多个文件

- x-www-form-urlencoded

   就是`application/x-www-from-urlencoded`,会将表单内的数据转换为键值对，比如,`name=Java&age = 23`

- raw

  可以上传任意格式的文本，可以上传text、json、xml、html等

- binary

   相当于`Content-Type:application/octet-stream`,从字面意思得知，只可以上传二进制数据，通常用来上传文件，由于没有键值，所以，一次只能上传一个文件

- `multipart/form-data`与`x-www-form-urlencoded`区别
  1. multipart/form-data：既可以上传文件等二进制数据，也可以上传表单键值对，只是最后会转化为一条信息
  2. x-www-form-urlencoded：只能上传键值对，并且键值对都是间隔分开的