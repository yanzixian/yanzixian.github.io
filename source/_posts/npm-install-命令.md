---
title: npm install 命令
date: 2020-04-08 09:36:28
categories: vue
tags:
---

### NPM

NPM是随同Node.js一起安装的包管理工具

#### npm install安装包的4种形式：

###### `npm install module_name`

将包安装到项目的node_modules目录下，但不会将包依赖写入package.json文件的dependencies或devDependencies节点中，因此运行`npm install`初始化一个项目时**不会**下载安装安装该依赖包

###### `npm install -g module_name`

全局安装依赖包，具体安装到磁盘哪个位置，要看`npm config prefix`的位置；

不会安装到node_modules目录下，不会将依赖写入dependencies和devDenpendencies节点中；

运行`npm install`初始化项目**不会**下载安装该依赖包

###### `npm install -save module_name`或`npm install -S module_name`

将包安装到node_modules目录下，并将依赖写入package.json文件的dependencies节点中

运行`npm install`初始化项目**会**下载安装该依赖包

运行npm install --production或者注明NODE_ENV变量值为production时，**会**自动下载包到node_modules目录中

###### `npm install -save-dev module_name`或`npm install -D module_name`

将包安装到node_modules目录下，并将依赖写入package.json文件的devDependencies节点中

运行`npm install`初始化项目**会**下载安装该依赖包

运行npm install --production或者注明NODE_ENV变量值为production时，**不会**自动下载包到node_modules目录中

**注：**

- package.json文件描述了项目的相关信息以及所有相关依赖包
- "dependencies"节点描述了项目在生产环境中使用到的所有依赖包，是项目运行是必须的依赖
- "devDependencies"节点描述了项目在开发时使用的所有依赖，这些依赖在项目部署后是不需要的