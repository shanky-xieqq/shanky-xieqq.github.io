---
layout: post
title:  "如何在原生微信小程序中使image图片自适应"
categories: weApp
tags: 微信小程序 minApp
---

* content
{:toc}

在最近的小程序的开发过程中，碰到一个这样的问题，后台传回了一个图片列表，我需要根据这些图片的大小自适应的循环展示。大家都知道在小程序中不像H5这样只给img设置宽度，高度它自己会适应。在小程序中，image标签必须要有宽度和高度，宽度直接100%就好了，问题是高度。
				



##### 步骤一：获取图片宽高
                
从小程序的API文档里我们可以看到，当image标签加载的时候，bindload方法会返回一个`event.detail = {height, width}`参数，由此我们可以获取到图片的宽度和高度，我们给循环的image列表加上bindload="imgLoad"。

```js
imgLoad:function(e){
    // 重新计算图片宽高
    util.imgLoadCalWidth({
      key: 'detailImgWidth['+e.target.dataset.index+']',
      target: this,
      width: e.detail.width,
      height: e.detail.height,
    });
  //获取图片的原始长宽
  var originalWidth = e.detail.width;
  var originalHeight = e.detail.height;
  var padding = 0;//这里设置左右两边的padding值
  var windowWidth = app.globalData.systemInfo.windowWidth-2*padding;//获取需要的设置的图片宽度
  //判断按照哪种方式进行缩放
  if (originalWidth > windowWidth) {//在图片width大于手机屏幕width时候
    originalHeight = (windowWidth * originalHeight) / originalWidth;
    originalWidth = windowWidth;
  } else {//否则展示原来的数据
    originalWidth = originalWidth;
    originalHeight = originalHeight;
  }
  this.setData({
    ['detailImgWidth['+e.target.dataset.index+']'+'.height']: originalHeight,
    ['detailImgWidth['+e.target.dataset.index+']'+'.width']: originalWidth,
  });
  }
```    

这样就能通过原图片的宽高比例计算出合适的图片宽高了。

##### 步骤二：优化，提取该方法

为了方便在我们需要的地方使用，我们将改方法提取到util.js文件里，稍微做一下封装

```js
/**
 * @param e.target  页面对象
 * @param e.key     参数设置key
 * @param e.width   图片原始宽度
 * @param e.height  图片原始高度
 * @param e.padding 图片padding值 默认0
 */
const imgLoadCalWidth = (e) =>{
  if (!e || !e instanceof Object) return;

  // //获取图片的原始长宽
  var originalWidth = e.width;
  var originalHeight = e.height;
  var padding = e.padding || 0;
  var windowWidth = app.globalData.systemInfo.windowWidth-2*padding;
  //判断按照哪种方式进行缩放
  if (originalWidth > windowWidth) {//在图片width大于手机屏幕width时候
    originalHeight = (windowWidth * originalHeight) / originalWidth;
    originalWidth = windowWidth;
  } else {//否则展示原来的数据
    originalWidth = originalWidth;
    originalHeight = originalHeight;
  }
  e.target.setData({
    [e.key+'.height']: originalHeight,
    [e.key+'.width']: originalWidth,
  });

}
```

当然别忘了exports。

```js
module.exports = {
  imgLoadCalWidth:imgLoadCalWidth,
}
```

##### 步骤三：更改imgLoad方法

现在将event传过来的参数通过imgLoadCalWidth传过去。      
     

```js
  imgLoad:function(e){
    // 重新计算图片宽高
    util.imgLoadCalWidth({
      key: 'detailImgWidth['+e.target.dataset.index+']',
      target: this,
      width: e.detail.width,
      height: e.detail.height,
    });
  }
```

这样就完成啦，接下来在需要的地方只需要引入util文件并且调用util.imgLoadCalWidth就好了。








