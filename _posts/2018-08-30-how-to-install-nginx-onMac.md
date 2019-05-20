---
layout: post
title:  "如何在macOS中安装并配置nginx"
categories: nginx
tags: macOS hack linux
---

* content
{:toc}

最近把公司的办公电脑装了黑苹果，一直纠结怎么把原来win的开发环境搬上来，特别是静态服务器该怎么配置。

用原先在linux上的方法我觉得nginx应该比较好配置[因为原先是iis所以没办法]。那么就开始吧
				



#### mac安装nginx配置过程

##### 安装			

###### 1.打开终端		

###### 2.安装brew命令                 

```js
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```    

###### 3.安装nginx

```js
brew install nginx
```

###### 4.启动nginx

```js
sudo nginx
```

OK,nginx就安装好了，在浏览器访问的默认端口是8080        

在浏览器输入 http://localhost:8080/ 就能看到nginx在本计算机搭建的服务器              

8080是nginx自带的默认网站设置的端口     

现在我们自己来创建一个网站，设置端口和映射路径  

##### 配置

###### 6.配置网站和端口

首先当然是新建几个html静态页面，组成一个‘小网站’，并且复制小网站的pwd路径。     

然后打开终端，准备编辑nginx配置。     

```js
vim /usr/local/etc/nginx/nginx.conf
```

进入nginx.conf页面后，按 “i" 键进入编辑状态，自定义端口，和配置本地网站TanWeb, 注意设置访问权限( user root owner; )，不然等会访问网站会出现403错误      

![](https://images2018.cnblogs.com/blog/454511/201804/454511-20180413100851619-948024920.png)     

按esc键退出编辑状态，输入  :wq  保存退出nginx.conf页面  

我用的配置如下：

```js
server{
        listen       8899;
        server_name  localhost;
        location / {
            root    /Users/shanky/webDev/webDev; #本地网站文件夹路径
            index   index.shtml  index.html;#设置默认主页

        }
    }
```   

重启nginx      

```js
sudo nginx -s reload
```

在浏览器输入localhost加自定义的端口，就能访问到配置好的网站了，比如我的端口配置为5188， 则在浏览器输入http://localhost:5188/       

###### 7.配置目录列表访问权限

我们知道nginx访问时如果目录下面没有默认首页，那么会返回403 Forbidden的错误，表示没有权限访问。      

那么该如何来显示目录列表呢，配置很简单只需要在location中加一行 `autoindex on;` 即可显示，这样默认显示的文件大小以字节为单位，并且时间和服务器时间相差8小时，所以一般应用中设置根据文件大小进行合适的显示，并且时间显示服务器时间。      

```js
server{
        listen       8899;
        server_name  localhost;
        location / {
            autoindex on;#显示本地网站目录
            root    /Users/shanky/webDev/webDev; #本地网站文件夹路径
            index   index.shtml  index.html;#设置默认主页

        }
    }
```

这样就能访问目录了，如果想要每次打开都根据点击访问，把index选项删了即可。     

###### 8.配置shtml

因为公司有好几个项目是用shtml的模式写的，要#include进来，所以还必须配置shtml服务；     

配置shtml服务也很简单，location里加上下面代码就好了。  

```js
ssi on;
ssi_silent_errors on;
ssi_types text/shtml;
```

这样我的静态服务器就配置好了。    

```js
server{
        listen       8899;
        server_name  localhost;
        location / {
            ssi on;
            ssi_silent_errors on;
            ssi_types text/shtml;#ssi开启shtml文件访问
            autoindex on;#显示本地网站目录
            root    /Users/shanky/webDev/webDev; #本地网站文件夹路径
            index   index.shtml  index.html;#设置默认主页

        }
    }
```






