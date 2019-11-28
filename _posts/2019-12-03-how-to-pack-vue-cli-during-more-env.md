---
layout: post
title:  "vue-cli3的多环境打包配置"
categories: vue
tags: 脚手架 vue webpack
---

* content
{:toc}

最近把项目从cli2升级到cli3，研究了下vue-cli3下多环境的配置


		   					    
				




### 安装


```shell
npm install -g @vue/cli
```

因为我主要用yarn（宿舍网络太辣鸡都9102年了还1M的带宽）：

```shell
yarn global add @vue/cli
```

项目新建后它的结构是这样的：

![](/img/img20191203.png)

相比较vue-cli2的项目，缺少了build和config目录，另外static目录也没了，多了个public目录

配置文件需要自己写一个vue.config.js

可以参考官方文档：[https://cli.vuejs.org/zh/config/#vue-config-js](https://cli.vuejs.org/zh/config/#vue-config-js)


### 环境配置


* 打开package.json文件,它原本的启动脚本是这样的。

![](/img/img2019120302.png)

在scripts里面修改环境配置，可以根据自己的项目来修改对应的环境，见下图:

![](/img/img2019120303.png)

修改完之后在项目根目录创建对应的文件，文件名以.env开头后面跟上.name,例如：.env.test

![](/img/img2019120304.png)

在.env.test里写上自己想写的配置，见下图：

![](/img/img2019120305.png)


注意：需要以VUE-APP_开头，后面的名字可以随意。我这里设置了一个VUE_APP_URL 来存储对应的服务器API地址。

最后，执行npm run test 即可将.env.test 文件里的配置带入到项目中，

在Vue项目中可以使用process.env.Vue_APP_URL 来使用.env.test 里设置的Vue_APP_URL，

其他以Vue_APP_开头的设置也可以读取到。







   













