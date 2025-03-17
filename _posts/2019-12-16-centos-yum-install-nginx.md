---
layout: post
title:  "Centos7用yum安装并配置nginx"
categories: Linux
tags: Linux Nginx
---

* content
{:toc}

最近在腾讯云新人活动搞了个200+的三年1核2G的服务器，200+的三年要什么自行车，

先上个nginx署个名先，哈哈。


		   					    
				



### 安装


* 添加 nginx 到 yum 源中


```shell
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

* 安装 nginx （在吧nginx添加到 yum 源之后，就可以使用 yum 安装了）

```shell
sudo yum install -y nginx
```

* 启动 nginx

```shell
sudo systemctl start nginx.service
```

如果一切顺利的话，现在就可以通过域名或者 ip 访问了

* 设置 nginx 开机自启动

```shell
sudo systemctl enable nginx.service
```

* nginx 配置信息

  * 网站文件存放默认位置（Welcome to nginx 页面）

  `/usr/share/nginx/html`

  * 网站默认站点配置

  `/etc/nginx/conf.d/default.conf`

  * 自定义 nginx 站点配置文件存放目录

  `/etc/nginx/conf.d/`

  * nginx 全局配置文件

  `/etc/nginx/nginx.conf`

  * 启动 nginx

  `service nginx start`

  * 关闭 nginx

  `service nginx stop`

  * 重启 nginx

  `service nginx reload`


### 配置


* 进入 `/etc/nginx`目录下，打开`nginx.conf` 文件最下面有一句话`include /etc/nginx/conf.d/*.conf`; 表明`conf.d` 下的所有以`.conf`结尾的文件都属于`nginx`的配置文件

* 进入 `conf.d` 下，只有一个 `default.conf` 默认配置文件，`cp default.conf test.conf` 复制一份 `default.conf` 并改名为 `test.conf`

* `vim test.conf` 打开 `test.conf` (只复制前几行)


```shell
server {
       listen       80;
       server_name  localhost;
  
       #charset koi8-r;
       #access_log  /var/log/nginx/host.access.log  main;
  
       location / {
           root   /usr/share/nginx/html;
           index  index.html index.htm;
      }
      
 # ....... 省略中间的代码     
     
     }
```

#### 第一种配置方法

* 把`server_name`后的`localhost`改为自己的域名 比如：`www.baidu.com` 没有的话，填写自己的 ip 也行

* root 表示 网页的路径，改为自己的 项目的路径

* index 自然就是主页了,修改如下

```shell
server {
       listen       80;
       server_name  www.XXXX.com;
 
     #charset koi8-r;
      #access_log  /var/log/nginx/host.access.log  main;
  
       location / {
   #        root   /usr/share/nginx/html;
  #        index  index.html index.htm;
           root  /opt/tomcat/apache-tomcat-8.5.39/webapps/wenjuan;
           index login.jsp;
      }
      
      
 # ....... 省略中间的代码 
 
      }
```

* `find / -name nginx` 查找一下名为`nginx`的目录 有一个是 `/usr/sbin/nginx`,然后进入`/usr/sbin`,输入 `nginx -t`

检查`nginx`配置是否有问题，`nginx`配置即使有问题，`nginx`服务也能正常启动或重启，只是不按照你的配置工作而已

* 错误的话会有提示哪个文件第几行有问题，自行修改即可。

* `nginx`配置正确之后 重启`nginx systemctl restart nginx` 或者 `service nginx reload` 也行

* 然后浏览器访问你的域名(上面填写的ip的话，访问ip就好了)。然后你会发现页面404了

* 造成这个错误有两个原因，1是配置的时候未指定index，2是权限不足（不能访问你指定的目录），翻上面看下自己的配置，权限不足的问题

* 修改方法：打开 `/etc/nginx/nginx.conf`

```shell
user  nginx;
   worker_processes  1;
  
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;

#省略以下代码
```

* 一个简单的修改方法就是，吧第二行的user之后的 nginx 改为 root

#### 第二种配置方式

```shell
server {
       listen       80;
       server_name  www.junhui.pro;

 
       location / {
         proxy_pass http://127.0.0.1:8080/;#代理了服务器8080端口
      }
```

* 保存之后 在`/usr/sbin`下,输入 `nginx -t` 检查 `nginx`配置是否有问题，没有问题在重启`nginx`

* 如果还有其他什么问题，可以查看`nginx`的日志情况，在 `var/log/nginx` 下










   













