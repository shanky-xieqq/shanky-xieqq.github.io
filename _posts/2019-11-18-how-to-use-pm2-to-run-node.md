---
layout: post
title:  "如何使用PM2运行node.js服务"
categories: Linux
tags: Node Linux Centos
---

* content
{:toc}

最近用koa2写了个纯静态用于在后端接口出来之前模拟数据的项目，将它布置在自己的服务器上，

但是因为node.js是单进程，进程被杀死后整个服务就跪了，所以需要进程管理工具，不过pm2远远不止这些				          

		   					    
				




### 官方


* Github地址:  [https://github.com/Unitech/pm2](https://github.com/Unitech/pm2)

* 官网:        [http://pm2.keymetrics.io/](http://pm2.keymetrics.io/)


### 介绍


PM2 是一个带有负载均衡功能的 Node 应用的进程管理器。

当你要把你的独立代码利用全部的服务器上的所有 CPU，并保证进程永远都活着，0 秒的重载， PM2 是完美的。



### 特性


* 内建负载均衡（使用Node cluster 集群模块）
* 后台运行
* 0秒停机重载(维护升级的时候不需要停机).
* 具有Ubuntu和CentOS 的启动脚本
* 停止不稳定的进程（避免无限循环）
* 控制台检测
* 提供 HTTP API
* 远程控制和实时的接口API ( Nodejs 模块,允许和PM2进程管理器交互 )



### 安装


首先确保有node.js 的环境

```shell
npm install -g pm2
```

### 运行


```shell
pm2 start app.js --name my-api # 命名进程
```

### 其他运行方式


```shell
 pm2 start app.js -i max  # 根据有效CPU数目启动最大进程数目
 pm2 start app.js -i 3      # 启动3个进程
 pm2 start app.js -x        #用fork模式启动 app.js 而不是使用 cluster
 pm2 start app.js -x -- -a 23   # 用fork模式启动 app.js 并且传递参数 (-a 23)
 pm2 start app.js --name serverone  # 启动一个进程并把它命名为 serverone
 pm2 stop serverone       # 停止 serverone 进程
 pm2 start app.json        # 启动进程, 在 app.json里设置选项
 pm2 start app.js -i max -- -a 23                   #在--之后给 app.js 传递参数
 pm2 start app.js -i max -e err.log -o out.log  # 启动 并 生成一个配置文件，你也可以执行用其他语言编写的app  ( fork 模式):
 pm2 start my-bash-script.sh    -x --interpreter bash
 pm2 start my-python-script.py -x --interpreter python
```

### npm 运行


```shell
pm2 start npm -- start
```

创建一个进程并把它命名为 test

```shell
pm2 start npm --name test -- start
```

### 使用


```shell
 npm install pm2 -g     # 命令行安装 pm2 
 pm2 start app.js -i 4 #后台运行pm2，启动4个app.js 
                               # 也可以把'max' 参数传递给 start
                               # 正确的进程数目依赖于Cpu的核心数目
 pm2 start app.js --name my-api # 命名进程
 pm2 list               # 显示所有进程状态
 pm2 monit              # 监视所有进程
 pm2 logs               #  显示所有进程日志
 pm2 stop all           # 停止所有进程
 pm2 restart all        # 重启所有进程
 pm2 reload all         # 0秒停机重载进程 (用于 NETWORKED 进程)
 pm2 stop 0             # 停止指定的进程
 pm2 restart 0          # 重启指定的进程
 pm2 startup            # 产生 init 脚本 保持进程活着
 pm2 web                # 运行健壮的 computer API endpoint (http://localhost:9615)
 pm2 delete 0           # 杀死指定的进程
 pm2 delete all         # 杀死全部进程
```


### 参考


* [https://www.jianshu.com/p/7b7378deb3e5](https://www.jianshu.com/p/7b7378deb3e5)





   













