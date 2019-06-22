---
layout: post
title:  "webpack项目中使用yarn代替npm安装依赖包"
categories: vue
tags: vue webPack
---

* content
{:toc}

最近国内GFW越来越严重了，可能有什么大事要发生，VPS也进了黑名单再加上宿舍网络本来就不是特别好，好多npm的资源都down不下来，cnpm又经常出现莫名其妙的缺斤少两，据说是安装路径比较奇怪，packager识别有问题。所以想到了用yarn代替npm安装依赖包的方式解决。
				




### yarn的安装

* 可以通过Homebrew包管理工具安装Yarn            
```shell
brew install yarn
```         

* 因为我是mac，所以还可以用MacPorts安装        
```shell
sudo port install yarn
```


##### 设置镜像源

* 同cnpm类似，安装完成后需要设置一下镜像源，以便加速下载      
```shell
yarn config set registry https://registry.npm.taobao.org --global      
yarn config set disturl https://npm.taobao.org/dist --global     
```

* 如果遇到EACCES: permission denied权限错误，可以尝试在brew前面加上sudo超级管理员模式。

* 安装完yarn之后就可以用yarn代替npm了，例如用yarn代替npm install命令，用yarn add 某第三方库名代替npm install --save 某第三方库名。










