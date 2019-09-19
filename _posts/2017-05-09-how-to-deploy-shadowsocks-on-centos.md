---
layout: post
title:  "centos7部署shadowsocks"
categories: Linux
tags: centos shadowsocks Linux
---

* content
{:toc}

首先需要一台国外的VPS，推荐[kvmla](https://www.kvmla.com)有永久8折的优惠码。

VPS选择Centos7的版本，mini或者minimal都可以，服务器当然小点好。




#### 安装Shadowsocks

 在命令端输入下面命令并回车
 
```
	yum -y update
	yum install -y python-setuptools && easy_install pip
	pip install shadowsocks
	yum clean all
```

 `yum clean all`只是清空安装包和缓存，因为新装的服务器 `yum -y install`应该会更新很多包，所以这里清空一下缓存和安装好的安装包,执行这些命令后服务器端已经安装好Shadowsocks了。
 
#### 配置Shadowsocks

执行下面命令使用 vi 编辑器编辑Shadowsocks配置文件
 
```
 	vi /etc/shadowsocks.json
``` 

 将如下配置文本粘贴到Shadowsocks配置文件中去
 
```
{
	"server": "0.0.0.0",
	"server_port": 11520,
	"local_address": "127.0.0.1",
	"local_port":1080,
	"timeout":300,
	"password": "monkeyd",
	"method": "aes-256-cfb"
}
```

服务器端口号(server_port)可以随意（大于1024），本地端口默认1080（socks5的国际惯例）。加密方式也可以选其他的，aes256安全性比较高，但可能因此在加解密上花更多时间，增加延迟。

"server" 指服务器公网ip地址，此处为防止报错直接写成 0.0.0.0 ，"server_port" 指定连接接端口，"password" 指定连接密码，"method" 指定加密方式

有的服务器server地址必须写成 0.0.0.0 才OK，否则会遇到`socket.error: [Errno 99] Cannot assign requested address`错误！
 
#### 开防火墙
 
 执行以下命令打开firewalld防火墙端口
 
```
	firewall-cmd --zone=public --add-port=11520/tcp --permanent
	firewall-cmd --zone=public --add-port=11520/udp --permanent
	firewall-cmd --reload
```

头两条的add-port=11520是开启端口，按照配置文件里的添加即可，最后一条是重启firewall防火墙。
 
#### 启动服务
 
 执行下面命令启动Shadowsocks服务

```
 	ssserver -c /etc/shadowsocks.json -d start
```

 记住上这条命令，重启linux后要执行一遍！否则不能启动Shadowsocks
 
当然你也可以把它加到开机启动项里，这样就不用每次手动启动了。
 客户端的话（windows和移动端）直接按照上面设置的配置添加线路即可。