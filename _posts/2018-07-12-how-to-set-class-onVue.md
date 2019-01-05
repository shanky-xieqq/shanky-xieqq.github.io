---
layout: post
title:  "vue根据data中的值设置class和style"
categories: vue
tags: Javascript vue
---

* content
{:toc}

具体实现如下
				

#### 主体

##### class 动态赋值			

下面代码是动态拼接`element-menu`的代码片段		

```js
<div v-for="(item,index) in menuList" :key="index">
    <li style="font-size: 14px;" 
        @click="clickOneLeveMenu(item)" 
        :class="{'top_nav_active':(defaultVal==item.id+''),'top_nav_unactive':(defaultVal!= item.id+'')}">
        <i :class="item.logo">&nbsp;{{item.menuName}}</i>
    </li>
</div>

data(){
    return{
        defaultVal:'',
        menuList:[
            {id:'1',menuName:'首页',logo:'fa fa-home'},
            {id:'2',menuName:'系统设置'，logo:'fa fa-cog'}
        ],
    }
}
```

简而言之				


```html
<div :class="{className:true/false}"></div>
```
		
##### style动态赋值			

利用三元表单式赋值			

```js
<div:style="{'color':(item.id == flag ? 'red' :'blue')}">test</div>

data(){
    return{
        flag:true,
    }
}
```



