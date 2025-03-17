---
layout: post
title:  "让jekyll服务可以在局域网中访问"
categories: Jekyll
tags: Jekyll
---

* content
{:toc}

本站是基于jekyll和github pages搭建的，jekyll搭建自己的博客非常方便，通过jekyll serve -w可以持续监控文件的修改情况、编译成HTML并通过HTTP服务给本机（localhost）进行访问。但是当你使用虚拟机的时候，特别是centos虚拟机连图形界面都没有，这时候你必须把它上传到github然后通过github.io来访问，这样改一步传一步很麻烦，由于jekyll将地址绑定到了127.0.0.1，导致局域网的其它机器并不能访问它的服务。原先我也走了很多弯路甚至想搭建Apache服务器来让外机可以访问虚拟机的jekyll项目，但实际上只要改变运行jekyll的参数就可以了




## 更改参数

```
$ jekyll serve -w --host=0.0.0.0

Server address: http://0.0.0.0:4000/

Server running... press ctrl-c to stop.
```

这样就可以通过该机器的IP地址访问网站了，同个局域网下任何机子都可以。