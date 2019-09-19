---
layout: post
title:  "git撤销commit错误reset后的恢复以及忽略已被纳入版本管理的文件"
categories: Git
tags: Git
---

* content
{:toc}

git commit的时候可能commit了不想上传的文件，这个时候可以回退到某次commit。

错误的reset --hard后想恢复原本的代码，以及当你不小心提交了不该提交的文件后，发现之后再添加.gitignore文件没用时，异常尴尬。
				          
		   					    
				




### 撤销commit

* 首先git log查看commit历史：

```shell
commit 8f12ffebea0705d21c43f9ae697192437e873063
Author: Shanky <shanky1993@163.com>
Date:   Wed Sep 18 23:22:27 2019 +0800

    [edit]日常提交

commit 50ac439248228fe3b85d1d81385c58a1f9b826fd
Author: Shanky <shanky1993@163.com>
Date:   Wed Sep 18 18:40:12 2019 +0800

    [edit]学习koa
```

* 找到要撤回到的那次commit id，然后执行如下命令：

```shell
git reset --mixed：//此为默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息
git reset --soft：//回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可
git reset --hard：//彻底回退到某个版本，本地的源码也会变为上一个版本的内容
```

* 一般使用`git reset --soft` 命令，`git reset --hard`要慎用，因为这会将本地代码也退回到历史的`commit id`。

### 错误reset后的恢复

* 当git reset --hard错误地将本地代码退回到历史commit后，也可以使用如下方法恢复。

首先执行git reflog（reflog会记录所有HEAD的历史，也就是说当你做 reset，checkout等操作的时候，都会被记录在reflog中）：

![](/img/img20190919.png) 

然后仍然使用git reset --hard命令强制将本地代码退回到历史的commit 。


### 取消已经纳入版本管理的文件

* .gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。 正确的做法是在每个clone下来的仓库中手动设置不要检查特定文件的更改情况：

```shell
git update-index --assume-unchanged PATH    //PATH为要忽略的文件
```

当第一次commit了文件到Git上，后来增加.gitignore文件，已经被push过的文件，仍然会被提交。要解决这个问题，执行以下命令（如要删除提交temp目录）：

```shell
git rm --cached temp
```  


   













