---
title: Collection中的toArray
date: 2020-04-24 16:22:00
categories: java

---

#### 概述

Collection集合接口中的`toArray`方法用于将Colleciton集合转化为数组

> This method acts as bridge between array-based and collection-based-APIs.

有两种重载形式：

```
Object[] toArray();
```

```
<T> T[] toArray(T[] a);
```

#### `Object[] toArray()`

此方法返回一个包含Collection集合所有元素的数组；

若集合迭代器规定了元素返回顺序，此方法元素返回顺序与之相同；

根据源码注释：

> The returned array will be "safe" in that no references to it are maintained by this collection.  (In other words, this method must
> allocate a new array even if this collection is backed by an array).
> The caller is thus free to modify the returned array

即重新分配了一个数组空间用于存储返回的元素,而不是使用相同的引用,所以对于集合和数组的编辑操作两者之间不受影响(**此处需要注意与Arrays.asList()方法进行区分,该方法转化后的集合直接引用了数组,数组和集合的变化会同步**)

#### `<T> T[] toArray(T[] a)`

此方法作用与上类似,返回一个包含Collection集合所有元素的数组

不同之处在于:

- 可以指定返回数组的类型
- 若传入的指定数组参数`a`大小不小于集合的大小,则将集合元素置于参数a中返回,`a`中剩余位置的元素置为`null`(可以用于确定集合大小,此时必须知道集合元素中不包含`null`);若数组`a`大小较小,则重新分配一个新的数组

例如`toArray(new Object[0])`功能与`toArray()`完全相同



注:List,Set接口继承Collection接口,实现了List,Set接口的类如ArrayList,HashSet均实现了上述两种方法