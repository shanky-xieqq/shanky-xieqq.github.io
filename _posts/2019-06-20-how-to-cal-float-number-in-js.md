---
layout: post
title:  "javascript浮点型数字运算的奇怪问题"
categories: javascript
tags: javascript
---

* content
{:toc}

`37.5*5.5=206.08`(我在项目里用JS算出来是这样的一个结果，四舍五入取两位小数，理论上是206.25才对呀)				    
我先怀疑是四舍五入的问题，就直接用JS算了一个结果为：206.08499999999998					      
怎么会这样，两个只有一位小数的数字相乘，怎么可能多出这么小数点出来。				      
我Google了一下，发现原来这是JavaScript浮点运算的一个bug。    				
比如：`7*0.8` JavaScript算出来就是：5.6000000000000005		   					    
				




### 解决方法

* 网上找到了一些解决办法，就是重新写了一些浮点运算的函数。      

下面就把这些方法摘录下来：  

```js

//除法函数，用来得到精确的除法结果
//说明：javascript的除法结果会有误差，在两个浮点数相除的时候会比较明显。这个函数返回较为精确的除法结果。
//调用：accDiv(arg1,arg2)
//返回值：arg1除以arg2的精确结果
function accDiv(arg1,arg2){
  var t1=0,t2=0,r1,r2;
  try{t1=arg1.toString().split(".")[1].length}catch(e){}
  try{t2=arg2.toString().split(".")[1].length}catch(e){}
  with(Math){
  r1=Number(arg1.toString().replace(".",""))
  r2=Number(arg2.toString().replace(".",""))
  return (r1/r2)*pow(10,t2-t1);
  }
}

//给Number类型增加一个div方法，调用起来更加方便。
Number.prototype.div = function (arg){
  return accDiv(this, arg);
}

//乘法函数，用来得到精确的乘法结果
//说明：javascript的乘法结果会有误差，在两个浮点数相乘的时候会比较明显。这个函数返回较为精确的乘法结果。
//调用：accMul(arg1,arg2)
//返回值：arg1乘以arg2的精确结果

function accMul(arg1,arg2)
{
  var m=0,s1=arg1.toString(),s2=arg2.toString();
  try{m+=s1.split(".")[1].length}catch(e){}
  try{m+=s2.split(".")[1].length}catch(e){}
  return Number(s1.replace(".",""))*Number(s2.replace(".",""))/Math.pow(10,m)
}

//给Number类型增加一个mul方法，调用起来更加方便。
Number.prototype.mul = function (arg){
  return accMul(arg, this);
}

//加法函数，用来得到精确的加法结果
//说明：javascript的加法结果会有误差，在两个浮点数相加的时候会比较明显。这个函数返回较为精确的加法结果。
//调用：accAdd(arg1,arg2)
//返回值：arg1加上arg2的精确结果
function accAdd(arg1,arg2){
  var r1,r2,m;
  try{r1=arg1.toString().split(".")[1].length}catch(e){r1=0}
  try{r2=arg2.toString().split(".")[1].length}catch(e){r2=0}
  m=Math.pow(10,Math.max(r1,r2))
  return (arg1*m+arg2*m)/m
}

//给Number类型增加一个add方法，调用起来更加方便。
Number.prototype.add = function (arg){
  return accAdd(arg,this);
}

```         

要用的地方包含这些函数，然后调用它来计算就可以了    

比如要计算：`7*0.8` ，则改成 (7).mul(8)            				

其它运算类似，就可以得到比较精确的结果。













