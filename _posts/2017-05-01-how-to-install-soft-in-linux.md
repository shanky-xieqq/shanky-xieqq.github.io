---
layout: post
title:  "如何在linux里安装软件[以Chrome为例]"
categories: linux
tags: linux
---

* content
{:toc}

当你把linux环境布置好了之后，又有一个蛋疼的事情来了，一些IDE，或者是linux下的软件这么装？当然如果是centos只需要命令式的就好了，一般都只要apt-get一下就OK了，我这里说的是ubuntu或者deepin之类的图形界面式的，我是小白啦，只能从图形开始玩。




## 获取安装包

打开控制台执行以下命令获取Chrome安装包，以64位为例：

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

32位可以执行以下命令，只是链接不同而已：

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_i386.deb
```

之后根目录将会有一个deb的安装包，当然你如果嫌在linux获取下载包慢或者资源的问题，可以在物理机或者别的windows的机子上用类似迅雷的p2p下载器下载好，然后VM的话用tool工具共享过来，或者直接用微云之类的云盘中转过来，这样就不会慢了。


## 安装软件

确保自己在根目录~下，执行以下命令执行安装：

```
sudo dpkg -i google-chrome*; sudo apt-get -f install
```

接下来系统就会执行安装命令，可能会需要你确定以下，输入Y就可以了。


## 延伸命令

理论上该安装命令可以安装所有deb的安装包，只要根目录下有你需要的安装包，可以不用wget，用其他方式从云盘或者哪里搞过来，然后执行命令：

```
sudo dpkg -i xxxxxxx*; sudo apt-get -f install
```

把xxxxxxx换成安装包的开头几个单词就可以了。