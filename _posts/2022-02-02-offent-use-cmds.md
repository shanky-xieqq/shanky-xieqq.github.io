---
layout: post
title:  "常用命令集锦"
categories: Linux
tags: Javascript Linux Centos
---

* content
{:toc}

有些复杂的常用命令经常会忘记到处找，为避免花时间所以集中记录一下，方便查找







## git删除分支

```bash
# 本地分支
git branch -d [branchName]

# 远程分支
git push origin --delete [branchName]
```


## PM2后台启动程序

```bash
pm2 start npm --name "mybook" -- run start --watch  #看书小工具
pm2 start npm --name "nigeria" -- run start   #尼日利亚
pm2 start -x './frpc' -n frp -- -c ./frpc.ini  #启动FRP
```

## nohub后台启动程序

```bash
# 后台运行frp
nohup ./frpc -c ./frpc.ini >/dev/null 2>&1 &
nohup ./frps -c frps.ini >/dev/null 2>&1 &
```

## zerotier命令指南

```bash
#zerotier-cli用法指引
	zerotier-cli info			    #查看当前zerotier-one的信息
	zerotier-cli listpeers			#列出所有的peers
	zerotier-cli listnetworks		#列出加入的所有的网络
	zerotier-cli join 		        #加入某个网络
	zerotier-cli leave 		        #离开某个网络
	zerotier-cli listmoons			#列出加入的Moon节点
	zerotier-cli orbit   	        #加入某个Moon节点
	zerotier-cli deorbit  	        #离开某个Moon节点

 #zerotier命令
 sudo systemctl start zerotier-one.service #启动
 sudo systemctl stop zerotier-one.service #停止
 systemctl enable zerotier-one.service	#打开开机自启
 systemctl disable zerotier-one.service		#关闭开机自启
```

## kms激活指令

```bash
slmgr /skms 192.168.192.112
#192.168.192.112是KMS服务器
slmgr /ato
```

## 查询端口占用

```bash
netstat -anp | grep 1123
```

## selinux操作

```bash
getenforce #查询selinux状态
setenforce 0 #临时将selinux关闭
```

## ssh的key生成指令

```bash
ssh-keygen -t rsa -C "youremail@example.com"
```
