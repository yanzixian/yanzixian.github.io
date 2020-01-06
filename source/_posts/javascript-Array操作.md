---
title: javascript Array操作
date: 2020-01-06 14:04:28
categories: javascript
tags: Array
---

#### `Array.map()`

`Array.map()`方法返回一个新数组，数组中的元素为原始数组元素经过调用函数处理后结果，不会改变原来数组的元素

##### 基本用法

`array.map(function(currentValue,index,arr),thisValue)`

##### 示例

```javascript
//简单数组
const arr=[1,2,3,4,5]
//定义处理函数
const square=(num)=>{
	return num*num
}
const res=arr.map(cube)//[1,4,9,16,25]
/**
* 还可以写为
* const res = arr.map((num)=>{
* 	return num * num;
* })
* 函数遍历数组，处理数组的每一个元素
/
```

