---
categories:
- web前端
tags:
- vue
---



使用webpack打包一个vue前端项目，部署到服务器上后发现在本地正常的css样式，在服务器上没有相应的样式，似乎css样式并没有进行打包，最后发现是`scoped`的问题

```
<style scoped>/*不添加scoped，则该样式不会在服务器端起作用*/
.el-form-list .model-block-label {
  width: 6em;
  text-align: right;
}
.el-form-list .model-block-content p {
  width: auto;
  margin-left: 0;
  min-width: 300px;
}
</style>

```
不太明白为什么，`scoped`的作用就是限制样式的作用范围，但为什么本地没问题，部署到服务器上却有问题，有待进一步研究
