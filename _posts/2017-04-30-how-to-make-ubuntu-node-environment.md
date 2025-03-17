---
layout: post
title:  "如何利用nvm版本管理器配置Ubuntu的node环境"
categories: Node
tags:  Node JavaScript Linux
---

* content
{:toc}

关于linux如何配置node环境，因为node环境需要干净的系统，所以linux是再好不过了，windows一直有不知名的bug，而且网上的node教程大多是面向linux环境的，所以配置一个linux的node环境是最好的，当然你可以用一个VM虚拟机配置linux，新手推荐Ubuntu甚至deepin都可以，图形化，方便有效，后期习惯了再去用centos，cenos主要是命令式的方便学习，不适合初学者，最起码不适合不熟悉linux的孩子。




## 安装

首先安装linux，我是用的ubuntu的VM虚拟机，装好后执行以下命令安装nvm：

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash
```

如果系统说没有找到curl，请执行以下命令安装，那直接按照系统说的执行就好了，笔者是已经配置好了所以忘记那个命令了哈哈哈。


## 更改nvm源

nvm安装后如果你直接执行`nvm install v6.0.0`（比方说我要安装6.0.0版本）你会发现安装超慢，所以要把安装源该成阿里巴巴的源：

命令`ll -a`找到根目录下的.bashrc文件
为了以防万一先执行以下命令备份.bashrc文件：

```
cp .bashrc .bashrc_back
```

这样你就备份了.bashrc文件，如果出问题直接把这两个名字换下就回来了。

接着安装vim编辑器，执行：

```
sudo apt-get install vim
```

`vim .bashrc`打开.bashrc文件，往文件末端添加如下代码，vim的修改方式请自行百度咯，打开之后i进编辑模式，改好ctrl+c退出编辑模式然后shirft+ZZ保存就可以了。

```
# nvm
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
```

更改完后退出保存，然后执行`source .bashrc`命令，这样更改就生效了。


## 安装node

利用nvm版本管理器：
- nvm ls 查看已经安装的版本
- nvm ls-remote 查看可用的版本
- nvm install v6.0.0 安装v6.0.0版本
- nvm alias default 6.0.0 设置6.0.0为默认版本


## nvm常用命令

- nvm install <version> ## 安装指定版本，可模糊安装，如：安装v4.4.0，既可nvm install v4.4.0，又可nvm install 4.4
- nvm uninstall <version> ## 删除已安装的指定版本，语法与install类似
- nvm use <version> ## 切换使用指定的版本node
- nvm ls ## 列出所有安装的版本
- nvm ls-remote ## 列出所以远程服务器的版本（官方node version list）
- nvm current ## 显示当前的版本
- nvm alias <name> <version> ## 给不同的版本号添加别名
- nvm unalias <name> ## 删除已定义的别名
- nvm reinstall-packages <version> ## 在当前版本node环境下，重新全局安装指定版本号的npm包


## 附：设置npm源

执行以下命令查看npm当前源：

```
npm config -g get registry
```

设置成淘宝镜像：

```
npm config -g set registry https://registry.npm.taobao.org
```
