---
categories:
- web前端
tags:
- vue
---



> 一个对象，键是需要观察的表达式，值是对应回调函数。值也可以是方法名，或者包含选项的对象。Vue 实例将会在实例化时调用 $watch()，遍历 watch 对象的每一个属性。
```js
var vm = new Vue({
  data: {
    a: 1,
    b: 2,
    c: 3,
    d: 4,
    e: {
      f: {
        g: 5
      }
    }
  },
  watch: {
    //键为表达式 a，值为回调函数
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // 方法名
    b: 'someMethod',
    // 该回调会在任何被侦听的对象的 property 改变时被调用，不论其被嵌套多深
    c: {
      handler: function (val, oldVal) { /* ... */ },
     //deep属性用于深度监听对象的属性；
     //若c为一个对象，有多个属性，当监听其中属性的变化时，就需要使用deep属性；
     //deep属性对于性能影响较大，修改任何一个属性都会触发监听
     //若不使用deep属性，只能监听到c这个引用的变化，即c指向一个新的对象时，监听才会生效，回调函数才会被调用
      deep: true      
    },
    // 该回调将会在侦听开始之后被立即调用
    d: {
      handler: 'someMethod',
    // immediate属性用于值开始被watch时，即watch中声明对某表达式的监听时就执行handler函数，而不是当表达式值改变时才执行
      immediate: true
    },
   // 三种值的写法，分别为方法名，回调函数，对象
    e: [
      'handle1',
      function handle2 (val, oldVal) { /* ... */ },
      {
        handler: function handle3 (val, oldVal) { /* ... */ },
        /* ... */
      }
    ],
    // watch vm.e.f's value: {g: 5}
    // 此类写法具体监听对象中的一个属性，不需要deep属性进行深度监听，
    'e.f': function (val, oldVal) { /* ... */ }
  }
})
vm.a = 2 // => new: 2, old: 1
```
> 注意，不应该使用箭头函数来定义 watcher 函数 (例如 searchQuery: newValue => this.updateAutocomplete(newValue))。理由是箭头函数绑定了父级作用域的上下文，所以 this 将不会按照期望指向 Vue 实例，this.updateAutocomplete 将是 undefined。
