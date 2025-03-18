---
layout: post
title:  "Centos安装docker以及docker-compose"
categories: Docker
tags: Docker Linux
---

* content
{:toc}

记录一下Centos7中安装docker和docker-compose的方法







1、yum更新

\# sudo yum update

2、如果安装docker旧版本，旧版本的卸载

# sudo yum remove docker docker-common docker-selinux docker-engine

3、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

\# sudo yum install -y yum-utils device-mapper-persistent-data lvm2

4、设置yum源

# sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo


5、可以查看所有仓库中所有docker版本，并选择特定版本安装

\# yum list docker-ce --showduplicates | sort -r

6、安装稳定版本

\# sudo yum install docker-ce


（安装特定版本）sudo yum install <FQPN\>  # 例如：sudo yum install docker-ce-17.12.0.ce  
  
7、启动并加入开机启动

\# sudo systemctl start docker  
\# sudo systemctl enable docker

8、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)


9、安装docker-compose  
\# sudo pip install -U docker-compose  
  

pip在centos也没有，如下处理

1.查看是否安装依赖包，没安装先安装：

sudo yum install epel-release

2.更新文件库

sudo yum -y update

3.安装pip

sudo yum -y install python-pip