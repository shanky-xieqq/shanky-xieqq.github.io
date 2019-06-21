---
layout: post
title:  "IE8及以下input的placeholder兼容"
categories: IE
tags: IE 兼容性
---

* content
{:toc}

最近的项目中遇到了需要兼容到IE8的placeholder，稍微记录一下以便以后或者需要的童鞋使用。

网上很多关于IE8placeholder的例子，有两个input重叠的、有使用vlue的，有用span替代placeholder的..很多，我是因为要用到Angular的表单验证，所以想稍微整合一下，整成自己所需要的。




#### Html

首先table是做表单最方便的东西了，所以这里就用一个简单的table做个小表单,为方便写样式和js加两个class。

```html
<table>
	<tr>
		<td>账户：</td>
		<td>
			<label class="inputs">
				<input type="text" placeholder="请输入账户" />
				<span class="iePlaceholder"></span>
			</label>
		</td>
	</tr>
	<tr>
		<td>密码：</td>
		<td>
			<label class="inputs">
			<input type="password" placeholder="请输入密码" />
			<span class="iePlaceholder"></span>
			</label>
		</td>
	</tr>
	<tr>
		<td>邮箱：</td>
		<td>
			<label class="inputs">
			<input type="email" placeholder="请输入邮箱" />
			<span class="iePlaceholder"></span>
			</label>
		</td>
	</tr>
</table>
```

html有了，接下来就是写那个隐藏placeholder的样式，因为要和原先的placeholder重叠，所以必须先`重置CSS`(见本文末)，这样在所有浏览器中才会一致。

#### Css

```css
input{border: 1px solid #ccc;font-size: 14px;padding: 4px 0px;line-height: 24px;height: 24px;}
.inputs{position: relative;}
.inputs .iePlaceholder{position: absolute;top: 0px;font-size: 14px;top: 2px;left: 1px;color: #a9a9a9;cursor: text;display: none;}
```

#### JS

我所用的js是基于jQuery的，所以在添加js前记得先引用jQuery哦。

```javascript
function placeholder(el) {
	//判断是否未显示placeholder
	function isplaceholder() {
		var input = document.createElement("input");
		return "placeholder" in input;
	}
	var $el = $(el);
	if(isplaceholder() == false && !('placeholder' in document.createElement('input'))) {
		$(".iePlaceholder").css("display","block");
		$('input[placeholder],textarea[placeholder]').each(function() {
			var that = $(this),
				text = that.attr('placeholder'),
			pans = $(this).parent("label").find("span.iePlaceholder");
			if(that.val() === "") {
				spans.html(text);
			}
			that.focus(function() {
					if(spans.html() == text) {
						spans.html("");
					}
				})
				.blur(function() {
					if(that.val() === "") {
						spans.html(text);
					}
				});
		});
	}
}
placeholder(".iePlaceholder");
```

#### 附录：重置css

这个重置Css是我从一个前辈那里拿过来的，一些细节可以自己添加自己修改，所谓自己需要的才是最好的嘛。

```css
/*初始化样式*/
html { -ms-text-size-adjust: 100%; -webkit-text-size-adjust: 100%; }
body { line-height:1.4; }
body, input, textarea, select { font-size:16px; color:#333; font-family:arial,"microsoft yahei",Verdana,Geneva,sans-serif; }
body, h1, h2, h3, h4, h5, h6, p, ul, ol, dl, dt, from { margin:0; font-weight:normal; }
table { border-collapse:collapse; border-spacing:0; }
ul, ol, dl, dt { padding:0; list-style-type:none; }
input { border:0; margin:0; padding:0px; background:none; height:20px; line-height:20px; border-radius:3px; }
input[type="checkbox"], input[type="radio"] { margin-right:5px; height:14px; line-height:14px; }
a{ color:#333; text-decoration:none; }
a:hover{ color:#096EA2; text-decoration:none; }
a img{ border:0; }
.fl{ float:left !important; }
.fr{ float:right !important; }
.clearfix{ zoom:1; }
.clearfix:before, .clearfix:after { content:""; display:table; }
.clearfix:after { clear:both; }
.clearfix { zoom:1; }
.clear { clear:both; display:block; overflow:hidden; visibility:hidden; width:0; height:0; }
article, aside, details, figcaption, figure, footer, header, main, menu, nav, section, summary { display: block; }
audio, canvas, progress, video { display: inline-block; }
audio:not([controls]) { display: none; height: 0; }
progress { vertical-align: baseline; }
```