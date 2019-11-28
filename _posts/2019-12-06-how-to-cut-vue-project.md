---
layout: post
title:  "vue-cli3打包太大首页加载过时间长的解决方案"
categories: vue
tags: 脚手架 vue webpack
---

* content
{:toc}

最近用vue-cli3构建一个小型的签到活动系统，完工后build发现一个chunk-vendors包就达到1.5MB，

加上其他一些资源文件，首页的下载总共大小快要3M，测试给的第一个反馈就是首屏慢的一批，必须优化。


		   					    
				




### 路由懒加载


结合Vue的异步组件再结合webpack的代码分割，我们可以轻松的实现路由懒加载。

** ️vue-cli3模式就使用了Babel，我们需要添加 `syntax-dynamic-import` 插件，才能使 Babel 可以正确地解析语法。


```js
// 安装插件 syntax-dynamic-import
cnpm install --save-dev @babel/plugin-syntax-dynamic-import

// 修改babel.config.js
module.exports = {
  "presets": [
    "@vue/app"
  ],
  "plugins": [
    [
      "component",
      {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
      },
      "syntax-dynamic-import"
    ]
  ]
}

// 修改路由组件的加载（router/index.js）
{
  path: '/',
  name: 'home',
  component: resolve => require(['pages/Home'], resolve)
}
```

使用了路由懒加载，已经把原来的chunk-vendors降到了1.1MB，但加载1M还是太大。那继续优化


### 服务器开启Gzip


Gzip是GNU zip的缩写，顾名思义是一种压缩技术。

它将浏览器请求的文件先在服务器端进行压缩，然后传递给浏览器，浏览器解压之后再进行页面的解析工作。

在服务端开启Gzip支持后，我们前端需要提供资源压缩包。

通过`CompressionWebpackPlugin`插件build提供压缩


```js
// 安装插件
cnpm i --save-dev compression-webpack-plugin

// 在vue-config.js 中加入
const CompressionWebpackPlugin = require('compression-webpack-plugin');
const productionGzipExtensions = ['js', 'css'];
const isProduction = process.env.NODE_ENV === 'production';

.....
module.exports = {
....
  configureWebpack: config => {
    if (isProduction) {
      config.plugins.push(new CompressionWebpackPlugin({
        algorithm: 'gzip',
        test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'),
        threshold: 10240,
        minRatio: 0.8
      }))
    }
  }
}
```

现在build一下看看压缩包的大小，大概减少了三分之二（GZIP果然是战斗机）;

### 启用CDN加速


用Gzip已把文件的大小减少了三分之二了，加载速度从之前2.7秒多到现在的1.8秒多，

但这个还是得不到满足。那我们就把那些不太可能改动的代码或者库分离出来，

继续减小单个chunk-vendors，然后通过CDN加载进行加速加载资源。

```js
// 修改vue.config.js 分离不常用代码库
module.exports = {
  configureWebpack: config => {
    if (isProduction) {
      config.externals = {
        'vue': 'Vue',
        'vue-router': 'VueRouter',
        'moment': 'moment'
      }
    }
  }
}
// 在public文件夹的index.html 加载

<!-- CND -->
<!--
<script src="https://cdn.bootcss.com/vue/2.5.17-beta.0/vue.runtime.min.js"></script>
-->
<script src="https://cdn.bootcss.com/vue/2.5.17/vue.runtime.min.js"></script>
<script src="https://cdn.bootcss.com/vue-router/3.0.1/vue-router.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/moment.js/2.22.2/moment.min.js"></script>
```

^^^ 如果使用了饿了么的element，不要使用beta版CDN，不然在日期组件上会遇到坑， 建议使用自家或者收费的CDN，保证CDN的可靠性

^^^ 推荐个好用的CND：[https://cdnjs.com/](https://cdnjs.com/)

现在再build后文件大小和访问速度都再次进行了提升，通过CDN加载整个文档速度已经接近1秒。


### 修改uglifyOptions去除console来减少文件大小


```js
// 安装uglifyjs-webpack-plugin
cnpm install uglifyjs-webpack-plugin --save-dev

// 修改vue.config.js
  configureWebpack: config => {
    if (isProduction) {
      .....
      config.plugins.push(
        new UglifyJsPlugin({
          uglifyOptions: {
            compress: {
              warnings: false,
              drop_debugger: true,
              drop_console: true,
            },
          },
          sourceMap: false,
          parallel: true,
        })       
      )
    }
  }
```

如果代码中打了很log，这个优化还是有点效果的。

在实际部署中发现加了`uglifyjs-webpack-plugin`整个包反而更大了，可能姿势不对，所以换了个姿势。

#### 姿势二：

使用`babel-plugin-transform-remove-console`插件

```shell
npm i --save-dev babel-plugin-transform-remove-console
```

在babel.config.js中配置

```js
const plugins = [];
if(['production', 'prod'].includes(process.env.NODE_ENV)) {  
  plugins.push("transform-remove-console")
}

module.exports = {
  presets: [["@vue/app",{"useBuiltIns": "entry"}]],
  plugins: plugins
};
```

### 总结


首屏优化工作告一段落，通过以上四项的优化，已经很大的提升了首屏的加载速度。

如果想再进一步优化还是空间的，例如从代码层面去减少代码量（代码复用率）









   













