---
layout: post
title:  "产品列表页图片高度自适应"
categories: CSS
tags: CSS
---

* content
{:toc}


今天项目遇到个坑爹的问题，图片高度自适应，淘宝列表宫格类的那种，两列，怪就怪在一个同事的手机分辨率太高，不然都找不出这种bug，哈哈。




#### 一个简单的知识点

这个BUG依赖于一个基础却又容易混淆的css知识点：当margin/padding取形式为百分比的值时，无论是left/right，还是top/bottom，都是以父元素的width为参照物的。      
#### 高度自适应占位      
场景： 

![](https://image-static.segmentfault.com/227/529/2275295169-5684c3e3ad4ae_articlex) 

如上图所示，有这么一种用来放图片的容器，图片都是正方形（为了方便举例用正方形，实际上只要固定长宽比例即可）。      
在PC端好办，容器的宽高都写死是多少px，这样即使图片加载不出来容器都不会变型。但是在移动端，由于各机型分辨率相差太大，写死px是绝对不可能的，终究还得靠百分比来实现自适应：      

1.容器宽度设个50%吧，这样一行放俩容器，各占屏幕宽度一半，没问题。      
2.图片宽度设个100%取容器的宽度，没问题。      
3.容器高度没法设置啊，因为容器宽高的参照物不一样，而且需求是高度与宽度一致，所以无法通过为容器高度设置百分比来达成，那就只能靠内容高度撑开了。      
4.容器的内容高度就是图片的高度，若图片是正方形，则图片高度与图片宽度一致，也即与容器宽度一致，看起来没问题是吧？实际上，在浏览器把图片加载出来以前，图片的高度是零，那可就没办法把容器撑开了，如下图所示：  

![](https://image-static.segmentfault.com/398/468/3984686490-5684c88db26f8_articlex)    

这样一来，即使图片加载速度很快，容器在图片加载前后都会有一个变型的过程，也就是俗称的“闪烁”，而如果图片加载不出来，整体布局就更是难看了。    
现在问题已经出来了，就是如何做到不靠图片本身就能把容器的高度撑开。    

#### 设置容器的padding-bottom/top    
使用margin/padding的百分比值来解决自适应高度的关键在于：容器margin/padding的百分比参照物是父元素的宽度，而容器的width的百分比参照物也是父元素的宽度，俩属性参照物一致，那么想要把这俩属性的值统一起来就很简单了。     
优化方案是这样的：给容器设置padding-top/padding-bottom跟width一致的值（百分比）。    

```css
#container {
  width: 50%;  //父元素宽度的一半
  background-color: red;  //仅为了方便演示
}
.placeholder {
  padding-top: 50%; //与width: 50%;的值保持一致，也就是相当于父元素宽度的一半。
}
```   

```html
<div id="container" class="placeholder"></div>
```

结果，容器的视觉效果如下：  

![](https://image-static.segmentfault.com/202/815/2028157010-5684da3fca545_articlex)

容器的盒子模型如下： 

![](https://image-static.segmentfault.com/781/417/781417916-5684dc77e3766_articlex)

从盒子模型可以看出，虽然容器的内容高度为0，但由于有了跟内容宽度一致的padding，因此整体视觉效果上像是被撑开了。此方案浏览器兼容性很不错，唯一的缺陷是无法给容器设置max-height属性了，因为max-height只能限制内容高度，而不能限制padding（我原以为设置box-sizing: border-box;可以让max-height限制padding，不过亲测无效，明白的朋友麻烦告知一下原因）。    

#### 给子元素/伪元素设置margin/padding撑开容器

从上面的方案看出`max-height`失效的原因是容器的高度本来就是`padding`撑的，而内容高度为0，`max-height`无法起作用。那想要优化这一点，唯一的方法就是利用内容高度来撑开而非`padding`，这个方案跟消除浮动所用的方案非常相似：给容器添加一个子元素/伪元素，并把子元素/伪元素的`margin/padding`设为100%，使其实际高度相当于容器的宽度，如此一来，便能把容器的高度撑至与宽度一致了。由于添加子元素与HTML语义化相悖，因此更推荐使用伪元素`(:after)`来实现此方案。    

```css
#container {
  width: 50%;
  position: relative;
  background-color: red;
  overflow: hidden;  //需要触发BFC消除margin折叠的问题
}
.placeholder:after {
  content: '';
  display: block;
  margin-top: 100%; //margin 百分比相对父元素宽度计算
} 
```

```html
<div id="container" class="placeholder"></div>
```

此时视觉效果上与上一方案无异，重点来看看此时容器的盒子模型：

![](https://image-static.segmentfault.com/104/560/1045604033-5684dc3a5b3b4_articlex)

可以看出，此时容器的内容高度与内容宽度一致，妈妈再也不用担心我无法通过max-height来限制容器高度了。    
另外，使用margin的话需要考虑margin折叠的问题，padding则无此烦恼。

#### 容器内部如何添加内容

上述方案只提及如何不依赖容器内容来撑开容器，那么，在撑开容器后，如何给容器添加内容（图片、文本等）呢？    
答案很简单，那就是利用`position: absolute;`

```css
#container {
  width: 50%;
  position: relative;
  background-color: red;
  overflow: hidden;  //需要触发BFC消除margin折叠的问题
}
.placeholder:after {
  content: '';
  display: block;
  margin-top: 100%; //margin 百分比相对父元素宽度计算
} 
img {
  position: absolute;
  top: 0;
  width: 100%;
}
```

```html
<div id="container" class="placeholder">
  <img src="http://img.arrayhuang.cn/product/miya-1060079/multiple/0.jpg@1e_415w_415h_1c_0i_1o_1x.jpg" />
</div>
```

效果如下：    

![](https://image-static.segmentfault.com/268/675/2686756191-5684e151bbd7a_articlex)

#### 总结

自适应的精髓在于宽度，`margin/padding`设置百分比弥补了元素高度无法自适应地与元素宽度保持一致的缺陷。


