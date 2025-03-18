---
layout: post
title:  "Docker常用命令大全"
categories: Docker
tags: Docker Linux
---

* content
{:toc}

有时候经常忘记docker的命令，简单汇总记录一下







## 一、启动类命令

```bash
systemctl start docker #启动 Docker
systemctl stop docker  #关闭 Docker
systemctl restart docker    #重新启动 Docker
systemctl enable docker     #设置 Docker 自启动
systemctl status docker     #查看 Docker 运行状态
docker version              #查看 Docker 版本号等信息
docker info                 #查看 Docker 信息,该命令还可以查看到有多少容器及其状态和镜像的信息
docker --help               #Docker 帮助,总体
docker run --help           #查看 docker run 的帮助文档
```

## 二、镜像类命令

```bash
docker images       #查看镜像
docker search [OPTIONS] 镜像名字   #搜索镜像
docker pull 镜像名   #拉取镜像
docker run 镜像名    #运行镜像
docker rmi 镜像名/镜像ID            #删除一个镜像
docker rmi -f 镜像名/镜像ID         #如果镜像在运行会报错，强制删除
docker rmi -f 镜像名/镜像ID 镜像名/镜像ID 镜像名/镜像ID     #删除多个镜像（镜像ID或镜像名用空格隔开）
docker rmi -f $(docker images -aq)  #删除全部镜像,其中 -a 表示显示全部镜像，-q 表示只显示镜像ID
docker load -i 镜像保存文件位置      #从文件加载镜像
docker save 镜像名/镜像ID -o 镜像保存位置和名字     #保存镜像
```

## 三、容器类命令

```bash
docker ps           #查看正在运行的容器
docker ps -a        #查看所有容器（包括停止的）
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"     #加格式化方式访问，格式会更加清爽
```

### 创建容器

常用参数：          
* `--name=NAME`：为容器指定名字为 NAME。
* `-d`：后台运行容器并返回容器ID，也即启动守护式容器（后台运行）。
* `-i`：以交互模式运行容器，通常与 -t 同时使用。
* `-t`：为容器重新分配一个伪输入终端，通常与 -i 同时使用。
* `-P`：随机端口映射，大写 P。
* `-p`：指定端口映射，小写 p。

创建并允许 Nginx 容器:

```bash
docker run -d --name nginx -p 80:80 nginx
```


```bash
docker start 容器名     #启动容器
docker stop 容器名      #停止容器
docker kill 容器名      #强制停止容器
docker rm 容器ID        #docker rm 容器ID
docker rm -f 容器ID     #强制删除
docker rm -f $(docker ps -a -q)     #删除多个容器
docker ps -a -q | xargs docker rm   #删除多个容器
docker logs 容器名      #查看容器日志
docker top 容器名       #查看容器内运行的进程
docker inspect 容器名   #查看容器内部细节
docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx     #创建容器数据卷挂载
docker volume ls        #查看数据卷
docker volume inspect 数据卷名      #查看数据卷详情
docker volume rm 数据卷名           #删除数据卷
```

## 四、网络类命令

```bash
docker network ls       #查看网络
docker network create 网络名        #创建网络
docker network inspect 网络名       #查看网络数据源
docker network rm 网络名            #删除网络
```

## 五、Docker Compose 命令

```bash
docker-compose -h       #查看帮助
docker-compose up       #启动所有服务
docker-compose up -d    #后台运行
docker-compose down     #停止并删除容器、网络、卷、镜像
docker-compose exec yml里面的服务id     #进入容器实例内部
docker-compose ps       #展示容器
docker-compose top      #展示进程
docker-compose logs yml里面的服务id     #查看容器输出日志
docker-compose config   #检查配置
docker-compose config -q    #检查配置，有问题才有输出
docker-compose start    #启动服务
docker-compose restart  #重启服务
docker-compose stop     #停止服务
```

## 六、其他

### 命令别名

修改 `/root/.bashrc` 文件：

```bash
vi /root/.bashrc
```

添加如下内容：

```bash
# .bashrc

# User specific aliases and functions
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias dps='docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"'
alias dis='docker images'

# Source global definitions
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi
```

保存并退出：

```bash
:wq
```

使别名生效：

```bash
source /root/.bashrc
```

