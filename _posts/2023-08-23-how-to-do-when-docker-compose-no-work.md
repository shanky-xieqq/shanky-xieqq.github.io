---
layout: post
title:  "Cenos7之重装docker-compose之后无法执行命令"
categories: Docker
tags: Docker Linux
---

* content
{:toc}

今天给测试机升级docker版本，重装docker-compose后发现命令无法执行了，记录一下解决方法。







### 背景

*   使用自动化脚本重装docker和docker-compose（但脚本中未对旧版本的docker-compose进行任何处理，比如卸载删除）
*   导致执行docker-compose命令时报了错，大多数是1，偶尔是2
    *   1、Cannot open self /usr/local/bin/docker-compose or archive /usr/local/bin/docker-compose.pkg
    *   2、Segmentation fault 段错误(跟内存混乱有关，相当于用A去执行B)

### 思路

*   发现在未安装docker-compose的服务器上可以正常执行
*   怀疑是docker-compose文件仍然是旧的版本，即未进行卸载旧docker-compose造成的
*   那么是直接去官网找到对应版本的docker-compose是否可以？

### 步骤

*   1、下载对应版本docker-compose, 我是最新版的，此时最新版链接：[https://github.com/docker/compose/releases/tag/1.25.0-rc4](https://github.com/docker/compose/releases/tag/1.25.0-rc4)
    
*   2、进入步骤1中的网页后搜索 docker-compose-Linux-x86\_64，复制该文件的链接地址
    
*   3、服务器上， cd /usr/local/bin/ ，wget [https://github.com/docker/compose/releases/download/1.25.0-rc4/docker-compose-Linux-x86\_64](https://github.com/docker/compose/releases/download/1.25.0-rc4/docker-compose-Linux-x86_64)
    
*   4、删除当前文件夹中docker-compose 文件，并将新下载的文件重命名为docker-compose
    
*   5 赋予权限 chmod +x /usr/local/bin/docker-compose, 如果你喜欢chmod -R 777， 那也可以。。。
    
*   6、 验证， 输入docker-compose --version ，此时出现版本信息， OK