---
layout: post
title:  "vue 使用html2canvas将DOM转化为图片"
categories: Vue
tags: Vue
---

* content
{:toc}

将DOM转化为图片是一个非常常见的需求，而自己手动转是非常麻烦的，于是找到了html2canvas这个插件，既是用得比较多的也是维护得比较好的一个插件。

版本比较多，这里主要介绍的是新版的最简单用法
				          
		   					    
				




### 安装


```shell
npm install html2canvas --save
```

* 现在最新的版本应该是1.0.0，另外还有一个比较经典的版本是0.5.0，网上有许多关于这个版本的bug说明。


### 使用


```html
<div class="imageWrapper" ref="imageWrapper">
    <img class="real_pic" :src="dataURL" />
    <slot></slot>
</div>
```

* slot里面是你需要转化为图片的DOM元素。

```js
data() {
    return {
        dataURL: ''
    }
},
```

* dataURL是最后转化出来的图片base64地址，放在img标签中即可展示。

methods:

```js
toImage() {
    html2canvas(this.$refs.imageWrapper,{
        backgroundColor: null
    }).then((canvas) => {
        let dataURL = canvas.toDataURL("image/png");
        this.dataURL = dataURL;
    });
}
```

* html2canvas的用法非常简单，不过1.0.0已经将写法改为了promise，在.then方法里获取canvas对象


### 常见bug


* 生成出来的图片有白色边框

在配置项里配置backgroundColor: null即可。


* 有图片显示不出来并有报错（一般是跨域的错）

这是最常见的一个bug，就是这个插件无法生成跨域了的图片，也看了官方文档配置了也百度了都没有好的办法，最后是让后端直接把跨域的图片转成base64，就完美解决了这个问题。


* 生成图片后会在原始DOM上覆盖而产生一个闪动的效果

先让生成的图片隐藏，等生成好以后再展示。（没有在手机上测试，效果不一定令人满意）



   













