---
layout: post
title:  "如何重写radio和checkbox"
categories: CSS
tags: CSS HTML
---

* content
{:toc}


产品和UI真会玩啊，这个要改那个要改，最近收到一个自定义checkbox和radio的需求，怎样写一个能读写数据又好看的样子呢？




#### 重写radio

```css
input[type=radio] {
	-webkit-appearance: none;
	appearance: none;
	width: 12px;
	height: 12px;
	margin: 0;
	margin-right: .2em;
	margin-bottom: 2px;
	cursor: pointer;
	vertical-align: middle;
	background: #fff;
	border: 1px solid #ccc;
	-webkit-border-radius: 1px;
	-moz-border-radius: 1px;
	border-radius: 1px;
	-webkit-box-sizing: border-box;
	-moz-box-sizing: border-box;
	box-sizing: border-box;
	position: relative;
	outline: none;
}

input[type=radio]:active {
	border-color: #ccc;
	background: #ebebeb;
}

input[type=radio] {
	-webkit-border-radius: 1em;
	-moz-border-radius: 1em;
	border-radius: 1em;
	width: 20px;
	height: 20px
}

input[type=radio]:checked {
	background: #fff;
	border: 2px solid #1ABC9A !important;
}

input[type=radio]:checked::after {
	content: '';
	display: block;
	position: relative;
	top: 3px;
	left: 3px;
	width: 10px;
	height: 10px;
	background: #1ABC9C;
	-webkit-border-radius: 1em;
	-moz-border-radius: 1em;
	border-radius: 1em
}
```

效果如图：

![](https://upload.wikimedia.org/wikipedia/commons/b/bf/Racio.png)


#### 重写checkbox

接下来是checkbox的

代码如下：

```css
input[type=checkbox] {
	-webkit-appearance: none;
	appearance: none;
	-moz-appearance: none;
	width: 16px;
	height: 16px;
	cursor: pointer;
	vertical-align: middle;
	background: #fff;
	border: 1px solid #ccc;
	-webkit-border-radius: 1px;
	-moz-border-radius: 1px;
	border-radius: 1px;
	-webkit-box-sizing: border-box;
	-moz-box-sizing: border-box;
	box-sizing: border-box;
	position: relative;
}

input[type=checkbox]:active {
	border-color: #c6c6c6;
	background: #ebebeb;
}

input[type=checkbox]:checked {
	background: #fff;
}

input[type=checkbox]:checked::after {
	content: url(../img/checked.png);
	display: block;
	position: absolute;
	top: -1px;
	right: -1px;
}

input[type=checkbox]:focus {
	outline: none;
	border-color: #4d90fe;
}

input[type=checkbox]:disabled {
	border: 1px solid #ccc;
	background: #e6e6e6;
}
```

效果如图：

![](https://upload.wikimedia.org/wikipedia/commons/b/b4/Checkbox-xieqq.png)

适用范围：

因使用了WebKit的CSS扩展属性，该方法适用于WebKit浏览器及移动端，微信或者小程序都是适用的。

注意：

checkbox的选中样式是一张背景图，可以通过Photoshop之类的去制作一个喜欢的，这里放上这里的用的以便需要的小伙伴。

它在这在这在这，![](https://upload.wikimedia.org/wikipedia/commons/1/15/Checked.png)

