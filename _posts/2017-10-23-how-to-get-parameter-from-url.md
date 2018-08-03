---
layout: post
title:  "从url后获取?之后的指定参数"
categories: JavaScript
tags: JavaScript
---

* content
{:toc}


在与后端的交互中要获取url上?后面的参数，于是写了一个复用function用于抓取?后面的指定参数。




#### 从url后获取?之后的指定参数

```js
function GetRequest() {
   var url = location.search; //获取url中"?"符后的字串
   var theRequest = new Object();
   if (url.indexOf("?") != -1) {
	  var str = url.substr(1);
	  strs = str.split("&");
	  for(var i = 0; i < strs.length; i ++) {
		 theRequest[strs[i].split("=")[0]]=(strs[i].split("=")[1]);
		  }
	   }
	   return theRequest;
	}
```

使用方法：

```js
var res = GetRequest();
var wechatcode=res['code'];//抓取code后面的参数
```


