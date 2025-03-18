---
layout: post
title:  "如何在Nuxtjs中做sitemap(站点地图)"
categories: Nuxt
tags: Nuxt Vue Javascript
---

* content
{:toc}





##### 第一步:安装`@/nuxt/sitemap`

```
npm install  @nuxtjs/sitemap
```

##### 第二步:在根目录`static`目录下新建`sitemap.js`

sitemap.xml文件的内容

> 这里也是可以是数组生成多个xml文件

```javascript
import axios from "axios";
const sitemap = {
  path: '/sitemap.xml', //生成的文件路径
  hostname: '自己网站的网址', //网站的网址
  cacheTime: 1000 * 60 * 60 * 24, //一天的更新频率，只在generate:false有用
  gzip: true, //生成.xml.gz的sitemap
  generate: false,
  exclude: ['/404', '/cart/api', '/confirmOrder/common/**', '/item/components/**','/category/minxinss','/category/components/**'], //排除不要的页面，这里的路径是相对于hostname
  defaults: {
    changefred: 'always',
    lastmod: new Date()
  },
  routes: async () => {
    let productList = await axios.post('商品列表接口地址', {
        categoryId: "",
        level: 0,
        pageNum: 1,
        pageSize: 20,
        sort: "DEFAULT"
      }

    ).then(res => {
      let proList = res.data.data.list;
      var siteArray = [];
      let siteObject = {};
      proList.forEach(element => {
        siteObject = {
          url: `/item/${element.id}.html`,
          changefred: 'daily',
          lastmod: new Date()
        }
        siteArray.push(siteObject);

      });
      return siteArray;
    })
    return productList ;

  },

  //   需要生成的xml数据，return 返回需要给出的xml数据
  // routes:()=>{
  //     const routes = [{
  //         url:"/",
  //         changefred:'always',
  //         lastmod:new Date()
  //     }]
  //     return routes
  // }

}
export default sitemap;

```

##### 第三步:在根目录`static`目录下新建`robots.txt`

> robots.txt文件可以指定爬虫只抓取指定的内容或者是禁止搜索部分内容。

```
# 该文件可以通过`你的网站域名/Robots.txt`直接访问

# User-agent作用：描述搜索引擎的名字，对于该文件来说至少药有一条user-agent记录，则该项的值设为*
User-agent: *
# Disallow:  描述不希望被访问到的一个url
Disallow: /bin/
Sitemap: 自己网站的域名/sitemap.xml
Sitemap: 自己网站的域名/sitemap.xml
Sitemap: 自己网站的域名/sitemap.xml
这里如果有大数据量的时候可以写多个sitemap
```

##### 第四步 在`nuxt.confing.js`中引用

```
import sitemap from './static/sitemap';
  modules: [
    '@nuxtjs/sitemap',
  ],
  sitemap:sitemap,
```

##### 第五步:要去谷歌搜索中心添加自己的站点地图

![Image 17: 在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/17a77b1add5ac18a5b3bee34b2b272f4.png#pic_center)

##### 第六步:查看效果

*   1，在浏览器中打开`自己网站的域名/sitemap.xml`看是否能直接打开，可以打开是xml文件就正确
*   2,在浏览器中打开`自己网站的域名/Robots.txt`看是否能直接打开，打开后文件如下所示

```
User-agent: *
Disallow: /404
Sitemap: 自己的域名/sitemap_1.xml
Sitemap: 自己的域名/sitemap_2.xml
```

这两个文件都可以访问成功就说明你的站点地图做好了