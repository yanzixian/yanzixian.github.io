---
title: vue ref和refs
date: 2020-04-07 16:07:46
categories: vue
tags:
---

### ref

根据官方文档

>`ref` 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 `$refs` 对象上。如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件

类型为`string`

在一个DOM元素上使用时

```html
<p ref="p">hello</p>
```

可以使用`vm.$refs.p`指向这个元素，进而可以对这个元素进行一些操作，如改变文本等

在一个子组件上使用时

```html
<child-component ref="child"></child-component>
```

可以使用`vm.$refs.child`指向这个子组件实例

> 另：当`v-for`用于元素或组件的时候，`ref`将是包含DOM或组件实例的数组
>
> 关于 ref 注册时间的重要说明：因为 ref 本身是作为渲染结果被创建的，在初始渲染的时候你不能访问它们 - 它们还不存在！`$refs` 也不是响应式的，因此你不应该试图用它在模板中做数据绑定

### vm.$refs

vue的一个实例属性，在上面已经使用过

一个对象，持有注册过 `ref` 特性 的所有 DOM 元素和组件实例，即拥有`ref`特性的DOM元素和组件都可以通过这个实例属性进行访问

举例：

```html
<template>
    <!-- $event是获取当前元素属性 -->
    <!-- ref相当于id -->
    <el-button ref='btn1'  @click="getname($event)">更改前</el-button>
</template>
<script>
export default {
  data() {
    return {
        name:'更改后'
    };
  },
  methods: {
      getname(val){
     
          this.$refs.btn1.innerText = name
      }
  },
  created() {
   
  },
  mounted() {
      
  }
};
</script>

```

可以起到动态更改按钮显示文字的效果（当然也可以通过`{{name}}`模版语法来进行更改）