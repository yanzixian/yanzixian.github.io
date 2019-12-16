---
categories:
- web前端
tags:
- vue
---



this是vue中的一个关键字，在不使用 `use strict`的非严格模式下，没有直接的挂载者（即调用者）的函数中this默认是指向window的，在使用`use strict`的严格模式下，没有直接的挂载者，this默认为`undefined`，以下均在非严格模式下讨论
_this为自己定义的一个变量，通常如下定义：

```
let _this=this;
或 var _this=this;
```
这里主要讨论this的指向问题:
####普通函数中的this:
普通函数中的`this`是指函数执行过程中，自动生成的一个内部对象，是指当前的对象，只在当前函数内部使用。（`this`对象是在运行时基于函数的执行环境绑定的：在全局函数（例如匿名函数）中，`this`指向的是`window`；当函数被作为某个对象的方法调用时，`this`就等于那个对象）。

在vue的官方文档中有这样一句话：
```
所有的生命周期钩子自动绑定 this 上下文到实例中，因此你可以访w问数据，对属性和方法进行运算。
这意味着你不能使用箭头函数来定义一个生命周期方法 (例如 created: () => this.fetchTodos())。
这是因为箭头函数绑定了父上下文，因此 this 与你期待的 Vue 实例不同，this.fetchTodos 的行为未定义。
```
######在vue组件中：
~~~
new Vue({
    data:{
            message:'hello'
     },
    created:function(){
           console.log(this.message)  //hello
    }
     methods:{
            output:function(){
             console.log(this.message)  //hello
            }
    }
~~~
vue组件或实例中，不管是生命周期函数还是自定义的方法中，this均指向当前vue实例
######在回调函数中：
具有代表性的为axios的回调
~~~
axios.get('/user', {
    params: {
    }
  })
  .then(function (response) {
    console.log(this.$route.name)
  })
  .catch(function (error) {
    console.log(this.$route.name)
  });
~~~
上面的代码中this指向为function，没有指向外部vue实例，而route为vue实例的属性，故无法获取到route，此时就要用到箭头函数或开头说的_this，使用_this如下：
~~~
let _this=this
axios.get('/user', {
    params: {
    }
  })
  .then(function (response) {
    console.log(_this.$route.name) //使用_this指向vue实例
  })
  .catch(function (error) {
    console.log(_this.$route.name) //使用_this指向vue实例
  });
~~~
####箭头函数中的this
**箭头函数**：箭头函数是es6新增的语法，出现的作用主要是使函数的书写变得简洁，还解决了this执行环境造成的一些问题。比如解决了匿名函数this的指向问题（匿名函数的执行环境具有全局性），包括setTimeout，setInterval等函数中使用this的问题。
**箭头函数的this定义**：箭头函数的this是在定义函数时绑定的，而不是在运行过程中绑定，即函数在定义时，this就继承了定义函数的对象。
有了箭头函数，上述代码还可如下：
~~~
axios.get('/user', {
    params: {
    }
  })
  .then(response => {    //箭头函数中this指向vue实例
    console.log(this.$route.name)
  })
  .catch(error => {      //箭头函数中this指向vue实例
    console.log(this.$route.name)
  });
~~~


