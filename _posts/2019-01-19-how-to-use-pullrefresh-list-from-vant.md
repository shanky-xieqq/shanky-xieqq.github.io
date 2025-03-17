---
layout: post
title:  "如何使用vant组件中的PullRefresh和List"
categories: Vue
tags: Vue Vant
---

* content
{:toc}

最近在修福报的过程中发现老的Vue项目中下拉刷新和上拉加载都是基于Iscroll写的，比较少更新而且有点乱，所以准备抽出一点时间把它们全部替换成最近流行的vant。
				




### vant组件中的PullRefresh和List

* 需求：项目首页经常性出现莫名其妙的卡死和下拉功能失败，重构下拉刷新和上拉加载。
* 解决方案：采用vant组件，List实现懒加载，PullRefresh实现下拉刷新



##### 初始化

* 首先在项目man.js里引入vant[因为该项目许多地方都有用到vant的组件所以采用全局引入]

```javascript
import Vant from 'vant'
import 'vant/lib/index.css';
```



##### Html

```html
<van-pull-refresh v-model="refreshing" @refresh="onRefresh">
    <van-list v-model="loading" :finished="finished" finished-text="没有更多了" @load="onLoad">
    ..//需要上拉加载和下拉刷新的内容
    </van-list>
</van-pull-refresh>
```


##### JS 
     
```javascript
export default{
    components:{},
    data(){
        return {
            finished:false,
            loading:false,
            refreshing:false
        }
    },
    methods:{
        onLoad:function(){
            //上拉加载以及第一次获取数据执行的方法
            //这里要注意的是vant的list把上拉刷新和页面首次加载的时候执行的方法合并在一起了，
            //也就是说onload里的方法在页面刷新的时候会执行第一次
        },
        onRefresh:function(){
            //下拉刷新执行的方法
        }
    }
}
```









