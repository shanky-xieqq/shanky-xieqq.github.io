---
layout: post
title:  "Vue-cli3在vsCode下愉快的调试"
categories: Vue
tags: Vue 脚手架 调试工具
---

* content
{:toc}

最近在把老项目vue-cli2搭建的做一下升级，新的cli3改版挺多，所以在调试方面做下记录；
				          
		   					    
				




### 配置

* 设置vue.config.js以便能够准确命中断点

```js
module.exports = {
  configureWebpack: {
    devtool: 'source-map'
  }
}
```

* 打开调试配置文件launch.json, 添加或修改以下配置:

```js
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "vuejs: chrome",
      "url": "http://localhost:8080",
      "webRoot": "${workspaceFolder}/src",
      "breakOnLoad": true,
      "sourceMapPathOverrides": {
        "webpack:///./src/*": "${webRoot}/*"
      }
    }
  ]
}
```

* 在终端运行程序，yarn serve, npm run serve,或者在vue-ui中启动均可。

* 点击debug绿色小三角，或者F5启动调试。



   













