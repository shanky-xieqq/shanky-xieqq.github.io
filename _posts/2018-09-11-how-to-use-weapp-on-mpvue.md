---
layout: post
title:  "如何在mpvue中正确的引用小程序的原生自定义组件"
categories: WeApp
tags: 微信小程序 MinApp Vue MpVue
---

* content
{:toc}

在实际开发中，我们通常希望使用已有的开源组件库来进行开发，这些开源组件库大多是基于原生自定义组件的方式写成，比如目前比较流行的Vant Weapp、iView Weapp等等。所以，在mpvue项目中如何引入并使用这些自定义组件，就成了必须了解的一个问题。
				



##### 步骤一：生成你的mpvue工程
                
通过vue-cli命令，我们先生成一个全新的mpvue工程代码：  

```js
vue init mpvue/mpvue-quickstart my-project
```    

然后进入该工程目录，通过npm安装依赖：

```js
cd my-project
npm install
```

##### 步骤二：下载小程序组件库

小程序的组件库有挺多，我们这里选用iVew Weapp作为示例。你可以直接去github把iView Weapp的代码下载下来，也可以用过npm来下载：

```js
npm i iview-weapp
```

下载完成后，到它的目录中寻找名为dist的目录，这里面存放的就是iView Weapp原生小程序自定义组件代码。

##### 步骤三：将组件库复制到工程的static目录下

你可以将上面提到的整个dist目录复制到你的mpvue工程下的static目录下（记得一定要是static目录，否则这些代码会被mpvue编译器错误的进行处理），然后给这个dist目录改个名字，比如叫iview。             

![](https://upload-images.jianshu.io/upload_images/3407939-c5ff282d0fbf3940.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/632/format/webp) 

##### 步骤四：为需要使用自定义组件的Page进行配置

我们知道，原生小程序开发中，我们如果要在Page中使用自定义的组件，则需要在该Page对应的`.json`配置文件中配置要使用的自定义组件。在mpvue中，我们也需要做等价的配置。    

比如，现在我要在`src/pages/index/index.vue`中使用iView中的`i-button`组件，那么就先要在`src/pages/index/main.json`（如果没有该文件，则新建一个）中进行如下配置：     

```js
{
  "usingComponents": {
    "i-button": "../../static/iview/button/index"
  }
}
```

##### 步骤五：在Page中使用自定义组件

经过上一步的配置，我们就可以开始在`src/pages/index/index.vue`中使用这个i-button组件了，就像这样：    

```js
<template>
  <div class="container">
    <i-button type="primary" @click="bindViewTap">这是一个按钮</i-button>
  </div>
</template>
```  

运行这个小程序，能看到如下的样子：

![](https://upload-images.jianshu.io/upload_images/3407939-aafb8458d8144f7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/744/format/webp)







