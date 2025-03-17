---
layout: post
title:  "在vue项目中封装的返回顶部组件"
categories: Vue
tags: Vue
---

* content
{:toc}

工作中遇到一个回到顶部的问题，刚开始没注意代码的低耦合直接写在页面上了，导致每个页面都多出一段将scroll置0和滚动的时候执行的代码，想来想去还是花了点时间把它们抽了出来，这样只要在页面中引入组件就好了。
				




### 组件：src/components/toTop.vue

* 需求：过多页面需要使用回到顶部按钮，需要降低代码耦合，提高开发效率。
* 解决方案：将其组件化



##### 初始化

* 首先在components文件夹中创建toTop.vue文件


##### 具体实现

```html
<template>
    <transition name="fade">
        <div class="toTop" v-if="shows" @click="goTop"><i class="el-icon-caret-top goTopIcon"></i></div>
    </transition>
</template>
```

```js
<script>
export default {
    name: 'toTop',
    data: function() {
        return {
            scrollTop: "",
            shows:false
        }
    },
    watch: {
        scrollTop(val) {
            if (this.scrollTop > 500) {
                this.shows = true;
            } else {
                this.shows = false;
            }
        }
    },
    methods:{
        handleScroll() {
            this.scrollTop =
                window.pageYOffset ||
                document.documentElement.scrollTop ||
                document.body.scrollTop;
            if (this.scrollTop > 50) {
                this.shows = true;
            }
        },
        goTop() {
            var timer = null,
                _that = this;
            cancelAnimationFrame(timer);
            timer = requestAnimationFrame(function fn() {
                if (_that.scrollTop > 0) {
                    _that.scrollTop -= 50;
                    document.body.scrollTop = document.documentElement.scrollTop =
                        _that.scrollTop;
                    timer = requestAnimationFrame(fn);
                } else {
                    cancelAnimationFrame(timer);
                    _that.shows = false;
                }
            });
        }
    },
    mounted() {
        window.addEventListener("scroll", this.handleScroll);
    },
    destroyed() {
        window.removeEventListener("scroll", this.handleScroll);
    },
}
</script>
```

```css
<style scoped>
.toTop{
    position: fixed;
    width: 72px;
    height: 72px;
    bottom: 30px;
    right: 20px;
    z-index: 1;
}
</style>
```


##### 引用
     
```javascript
<template>
  <div id="show">   
    <toTop></toTop>
  </div>
</template>
<script>
  import toTop from "@/components/toTop";
  export default {
    data() {
      return {
        data: []

      }
    },
    components:{
      toTop
    }
  }
</script>
```









