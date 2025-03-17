---
layout: post
title:  "vue项目移植报错以及解决办法"
categories: Vue
tags: Linux Vue
---

* content
{:toc}

vue项目移植报错以及解决办法

之前项目中遇到一个坑，把项目从win10系统复制到mac系统上开发，输入命令行 npm run start后会报类似以下错误： 				





#### 主体

```shell
ERROR in Missing binding H:\myWork\lvlvPro\lvlvPro\node_modules\node-sass\vendor\win32-ia32-48\binding.node
Node Sass could not find a binding for your current environment: Windows 32-bit with Node.js 6.x
Found bindings for the following environments:
  - Windows 64-bit with Node.js 6.x
This usually happens because your environment has changed since running `npm install`.
Run `npm rebuild node-sass` to build the binding for your current environment.
 @ ./src/assets/stylesheets/base.scss 4:14-127 13:2-17:4 14:20-133
ERROR in Missing binding H:\myWork\lvlvPro\lvlvPro\node_modules\node-sass\vendor\win32-ia32-48\binding.node
Node Sass could not find a binding for your current environment: Windows 32-bit with Node.js 6.x
Found bindings for the following environments:
  - Windows 64-bit with Node.js 6.x
This usually happens because your environment has changed since running `npm install`.
Run `npm rebuild node-sass` to build the binding for your current environment.
 @ ./src/assets/stylesheets/style.css 4:14-127 13:2-17:4 14:20-133
ERROR in Missing binding H:\myWork\lvlvPro\lvlvPro\node_modules\node-sass\vendor\win32-ia32-48\binding.node
Node Sass could not find a binding for your current environment: Windows 32-bit with Node.js 6.x
Found bindings for the following environments:
  - Windows 64-bit with Node.js 6.x
This usually happens because your environment has changed since running `npm install`.
Run `npm rebuild node-sass` to build the binding for your current environment.
 @ ./~/react-datepicker/dist/react-datepicker.css 4:14-106 13:2-17:4 14:20-112
```

此时运行按照提示执行 `npm rebuild node-sass --force` 命令   
 		
`hufeideMacBook-Pro:DrFarmerOnline hufei$ npm rebuild node-sass —force`				
然后 `npm run start`，启动服务，如果还是发现报错，就把当前的node版本更换为在win10开发环境下的node的版本即可


