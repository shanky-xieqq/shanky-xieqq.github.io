---
layout: post
title:  "纯CSS绘制三角形[各种角度]"
categories: CSS
tags: CSS
---

* content
{:toc}

我们的网页因为 CSS 而呈现千变万化的风格。这一看似简单的样式语言在使用中非常灵活，只要你发挥创意就能实现很多比人想象不到的效果。特别是随着 CSS3 的广泛使用，更多新奇的 CSS 作品涌现出来，今天尝试一下CSS三角形绘制方法。
				



##### 等边三角形-上
      

![](/img/201310290941121.jpg)

```css
#triangle-up {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-bottom: 100px solid red;
}
```    


##### 等边三角形-下


![](/img/201310290941322.jpg)

```css
#triangle-down {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-top: 100px solid red;
}
``` 


##### 等边三角形-左      
     

![](/img/201310290941433.jpg)

```css
#triangle-left {
    width: 0;
    height: 0;
    border-top: 50px solid transparent;
    border-right: 100px solid red;
    border-bottom: 50px solid transparent;
}
``` 

##### 等边三角形-右     
     

![](/img/201310290941534.jpg)

```css
#triangle-right {
    width: 0;
    height: 0;
    border-top: 50px solid transparent;
    border-left: 100px solid red;
    border-bottom: 50px solid transparent;
}
``` 

##### 直角三角形-上左
      

![](/img/201310290942025.jpg)

```css
#triangle-topleft {
    width: 0;
    height: 0;
    border-top: 100px solid red;
    border-right: 100px solid transparent;
}
```    


##### 直角三角形-上右


![](/img/201310290942136.jpg)

```css
#triangle-topright {
    width: 0;
    height: 0;
    border-top: 100px solid red;
    border-left: 100px solid transparent; 
}
``` 


##### 直角三角形-下左 
     

![](/img/201310290942227.jpg)

```css
#triangle-bottomleft {
    width: 0;
    height: 0;
    border-bottom: 100px solid red;
    border-right: 100px solid transparent;
}
``` 

##### 直角三角形-下右     
     

![](/img/201310290942328.jpg)

```css
#triangle-bottomright {
    width: 0;
    height: 0;
    border-bottom: 100px solid red;
    border-left: 100px solid transparent;
}
``` 








