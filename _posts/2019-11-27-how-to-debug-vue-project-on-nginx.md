---
layout: post
title:  "vue-cli项目在nginx下子页面刷新404"
categories: nginx
tags: nginx Linux vue
---

* content
{:toc}

最近把项目通过nginx反向代理后发现，只有index主目录进去后刷新没问题，

所有子目录子路由，在刷新后都报404			          

		   					    
				




### 解决方案


* 原因：这是因为目录刷新后没有在root下找到该文件，所以只要把目录重新指向index即可；

```shell
location / {
　　root  /；
　　index index.html;
　　try_files $uri $uri/ /index.html
}
```

如果不是在根目录下，有特定目录的话则：

```shell
location /xx/xx/ {
　　root  /；
　　index index.html;
　　try_files $uri $uri/ /xx/xx/index.html
}
```

附上官方vue-router的说明：(https://router.vuejs.org/zh-cn/essentials/history-mode.html)[https://router.vuejs.org/zh-cn/essentials/history-mode.html]


### 另外一个空白方案


vue打包后发布在nginx下，页面加载了js但是页面显示空白

这个问题是因为在配置router的时候没有带上自己的项目的目录，在配置router那里加上项目路径就可以了

直接写在router上：

```js
export default new Router({

  mode: 'history',
  base: '/xx/xxx',
  routes: [
    {
      path: '/',
      name: 'Login',
      component: signIn
    }
  ]
})
```

写成全局变量在配置文件里，以便发布

```js
export default new Router({

  mode: 'history',
  base:env.base_build_path,
  routes: [
    {
      path: '/',
      name: 'Login',
      component: signIn
    }
  ]
})
```

注：这个env.base_build_path就是配置文件里的一个全局变量，也是项目路径





   













