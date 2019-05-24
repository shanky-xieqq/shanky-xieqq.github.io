---
layout: post
title:  "VUEJS开发规范"
categories: vue
tags: vue
---

* content
{:toc}

最近在整VueJs相关的东西，因为产品那边有把项目H5化的想法，所以现在不忙就稍微熟悉一下，毕竟许久没写H5端了，一直在搞原生小程序，所以练练手，搞个规范。
				




### VUEJS开发规范

本文摘自segmentfault，感谢@一只特立独行的猫。原文地址->[VUEJS开发规范](https://segmentfault.com/a/1190000012610056 "VUEJS开发规范")
      
1. 基于组件化开发理解
2. 组件命名规范
3. 结构化规范
4. 注释规范
5. 编码规范



##### 基于组件化开发理解

- 什么是组件？

```
组件其实就是页面组成的一部分，好比是电脑中的每一个元件（如硬盘、键盘、鼠标），它是一个具有独立的逻辑和功能或界面，同时又能根据规定的接口规则进行相互融化，变成一个完整的应用。
页面只不过是这样组件的容器，组件自由组合形成功能完整的界面，当不需要某个组件，或者想要替换某个组件时，可以随时进行替换和删除，而不影响整个应用的运行。前端组件化的核心思想就是将一个巨大复杂的东西拆分成粒度合理的小东西。
``` 

- 组件化开发的好处

```cs
提高开发效率
方便重复使用
简化调试步骤
提升整个项目的可维护性
便于协同开发
使其高内聚，低耦合，达到分治与复用的目的。
```

- 组件化和模块化的区别

```cs
组件化是从产品功能角度进行分割，模块化是从代码实现角度进行分割，模块化是组件化的前提和基础。
```

- Vue组件化开发

```cs
单文件系统，样式局部作用域
基本组成结构：<template/> <script/> <style scoped/>
组件注册方式：1）公共组件全局注册 2）其余组件局部注册
```


##### 组件命名规范   
     
Vue官方文档给予以下说明：


```cs
当注册组件 (或者 prop) 时，可以使用 kebab-case (短横线分隔命名)、camelCase (驼峰式命名) 或 PascalCase (单词首字母大写命名)。
PascalCase 是最通用的声明约定而 kebab-case 是最通用的使用约定。
``` 

命名可遵循以下规则：

```cs
1、有意义的名词、简短、具有可读性
2、以小写开头，采用短横线分割命名
3、公共组件命名以公司名称简拼为命名空间(app-xx.vue)
4、文件夹命名主要以功能模块代表命名
同时还需要注意：必须符合自定义元素规范: 使用连字符分隔单词，切勿使用保留字。app- 前缀作为命名空间: 如果非常通用的话可使用一个单词来命名，这样可以方便于其它项目里复用。
```

##### 结构化规范 

- 基于Vue-cli脚手架的结构基础划分

```js
├── index.html                      入口页面
├── build                           构建脚本目录
│   ├── build-server.js                 运行本地构建服务器，可以访问构后的页面
│   ├── build.js                        生产环境构建脚本
│   ├── dev-client.js                   开发服务器热重载脚本，主要用来实现开发阶段的页面自动刷新
│   ├── dev-server.js                   运行本地开发服务器
│   ├── utils.js                        构建相关工具方法
│   ├── webpack.base.conf.js            wabpack基础配置
│   ├── webpack.dev.conf.js             wabpack开发环境配置
│   └── webpack.prod.conf.js            wabpack生产环境配置
├── config                          项目配置
│   ├── dev.env.js                      开发环境变量
│   ├── index.js                        项目配置文件
│   ├── prod.env.js                     生产环境变量
│   └── test.env.js                     测试环境变量
├── mock                            mock数据目录
│   └── hello.js
├── package.json                    npm包配置文件，里面定义了项目的npm脚本，依赖包等信息
├── src                             项目源码目录    
│   ├── main.js                         入口js文件
│   ├── App.vue                         根组件
│   ├── components                      公共组件目录
│   │   └── title.vue
│   ├── assets                          资源目录，这里的资源会被wabpack构建
│   │   ├── css                         公共样式文件目录
│   │   ├── js                          公共js文件目录
│   │   └── img                      图片存放目录
│   ├── routes                          前端路由
│   │   └── index.js
│   ├── store                           应用级数据（state）
│   │   └── index.js
│   └── views                           页面目录
│       ├── hello.vue
│       └── notfound.vue
├── static                          纯静态资源，不会被wabpack构建。
└── test                            测试文件目录（unit&e2e）
    └── unit                            单元测试
        ├── index.js                        入口脚本
        ├── karma.conf.js                   karma配置文件
        └── specs                           单测case目录
            └── Hello.spec.js
``` 

- vue文件基本结构

```html
<template>
    <div>
    <!--必须在div中编写页面-->
    </div>
</template>
<script>
    export default {
    components : {
    },
    data () {
        return {
        }
    },
    methods: {
    },
    mounted() {

    }
    }
</script>
<!--声明语言，并且添加scoped-->
<style lang="less" scoped>
</style>
```

- vue文件方法声明顺序

```cs
- components   
- props    
- data     
- created
- mounted
- activited
- update
- beforeRouteUpdate
- metods   
- filter
- computed
- watch
```

##### 注释规范
      

代码注释在一个项目的后期维护中显的尤为重要，所以我们要为每一个被复用的组件编写组件使用说明，为组件中每一个方法编写方法说明。
以下情况，务必添加注释

```cs
1.公共组件使用说明
2.各组件中重要函数或者类说明
3.复杂的业务逻辑处理说明
4.特殊情况的代码处理说明,对于代码中特殊用途的变量、存在临界值、函数中使用的hack、使用了某种算法或思路等需要进行注释描述
5.注释块必须以/**（至少两个星号）开头**/；
6.单行注释使用//；
```    

- 单行注释

```cs
普通方法一般使用单行注释// 来说明该方法主要作用
```

- 多行注释

```html
组件使用说明，和调用说明 
<!--公用组件：数据表格
    /**
    * 组件名称
    * @module 组件存放位置
    * @desc 组件描述
    * @author 组件作者
    * @date 2017年12月05日17:22:43
    * @param {Object} [title]    - 参数说明
    * @param {String} [columns] - 参数说明
    * @example 调用示例
    *  <hbTable :title="title" :columns="columns" :tableData="tableData"></hbTable>
        */
    -->
```


##### 编码规范


优秀的项目源码，即使是多人开发，看代码也如出一人之手。统一的编码规范，可使代码更易于阅读，易于理解，易于维护。尽量按照ESLint格式要求编写代码

```cs
1.使用ES6风格编码源码
    定义变量使用let ,定义常量使用const
    使用export ，import 模块化
2.组件 props 原子化
    提供默认值
    使用 type 属性校验类型
    使用 props 之前先检查该 prop 是否存在
3.避免 this.$parent
4.谨慎使用 this.$refs
5.无需将 this 赋值给 component 变量
6.调试信息console.log() debugger 使用完及时删除
``` 









