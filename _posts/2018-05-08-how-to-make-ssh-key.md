---
layout: post
title:  "客户端生成ssh的命令"
categories: Linux
tags: Linux
---

* content
{:toc}

经常会用到的客户端生成ssh的命令，留存一下




#### 主体

```linux
ssh-keygen -t rsa -C "youremail@example.com"
```

一般生成后在pc的我的文档-> .ssh -> id_rsa.pub里，    
win10是在C:\Users\Administrator\.ssh\id_rsa.pub里


