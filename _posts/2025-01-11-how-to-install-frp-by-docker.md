---
layout: post
title:  "基于Docker搭建frp内网穿透"
categories: Docker
tags: Docker Linux
---

* content
{:toc}

最近在整公司服务器的测试地址，因为有外网测试和访问的需求，又不想把代码丢云服务器，想部署在本地方便gitlab的脚本自动化部署，所以基于frp做了内网穿透功能，方便内网外网以及不同部门之间的访问。








连接拥有公网 IP 的服务器，在合适的位置创建`frps`目录作为工作空间

```shell
# 创建 frps 目录作为工作空间
$ mkdir frps

# 创建服务端配置文件
$ touch frps/frps.toml

# 编辑服务端配置文件
$ vim frps/frps.toml
```

服务端配置文件内容如下所示

```toml
# 服务器的公网IP
bindAddr = "服务器的公网IP"
# 与客户端建立连接的端口
bindPort = 7000

# 服务端控制面板
webServer.addr = "服务器的公网IP"
# 访问控制面板的端口号
webServer.port = 7500
# 控制面板的用户名和密码，暴露在公网的服务请使用严谨一些的用户名密码
webServer.user = "admin"
webServer.password = "123456"

# 配置服务端的鉴权，这里使用Token进行鉴权，客户端必须用指定的Token才可以与服务端建立连接，防止滥用
auth.method = "token"
auth.token = "gbfvzhsybvtybsibvuipqfnnvlkashfgiawug"

# 配置服务端只打印warn级别的日志，并将日志输出到指定目录（注意这个目录指向的是容器内的目录）
log.level = "warn"
log.to = "/opt/frps/frps.log"
```

配置文件编写完成后下载`fatedier/frps:v0.61.2`镜像，不同与网上流传的教程（他们啥版本都有），该镜像应该是原作者提供的，镜像仓库名称和作者 Github 名称一致，且该镜像会及时跟进软件版本，v0.61.2 是截止到本文发布时的最新的版本

```shell
# 下载Docker镜像，Docker网络很迷，下载失败也不要紧，后面会帮你解决
$ docker pull fatedier/frps:v0.61.2

# 启动服务端 frps，推荐网桥用 host 类型，将刚刚创建的工作空间目录映射到容器中并指定配置文件启动
$ docker run --name frps \
   --restart always \
   --network host \
   -e TZ=Asia/Shanghai \
   -v ./frps:/opt/frps \
   -d fatedier/frps:v0.61.2 -c /opt/frps/frps.toml
```

执行命令后如果容器正常运行，没有自动停止，就算启动成功了，日志文件空白属于正常现象，因为配置文件中设置了只打印 warn 级别日志，启动成功的 info 级别日志不会打印，通过容器运行状态判断是否启动成功即可

启动成功后可以通过之前配置的控制面板检查 frps 的状态，之前配置的是 7500 端口，这里进行访问测试，需要注意该容器是使用 host 网桥启动的，如果服务器中启用了防火墙需要放行之前配置的 7000 和 7500 端口


客户端搭建 frpc
-----------------------------------------------------------------------------------------------------------

如果你是 win 或移动端用户，请自行[访问发行页面](https://github.com/fatedier/frp/releases/tag/v0.61.2)下载合适的版本以及客户端软件，这里仍然以 Linux Docker 环境举例搭建 frpc 客户端

同之前流程相似，在合适的位置创建`frpc`目录作为工作空间

```shell
# 创建 frpc 目录作为工作空间
$ mkdir frpc

# 创建客户端配置文件
$ touch frpc/frpc.toml

# 编辑客户端配置文件
$ vim frpc/frpc.toml
```

客户端配置文件内容如下所示

```toml
# 与服务端建立连接，跟上面的配置要对应
serverAddr = "服务端IP地址"
serverPort = 7000

# 配置Token鉴权，要与服务端一致
auth.method = "token"
auth.token = "gbfvzhsybvtybsibvuipqfnnvlkashfgiawug"

# 配置日志信息
log.level = "warn"
log.to = "/opt/frpc/frpc.log"
```

配置文件编写完成后下载`fatedier/frpc:v0.61.2`镜像，强烈建议与服务端版本一致

```shell
# 下载Docker镜像，Docker网络很迷，下载失败也不要紧，后面会帮你解决
$ docker pull fatedier/frpc:v0.61.2

# 启动客户端 frpc，将刚刚的工作空间映射到容器中并指定配置文件启动
$ docker run --name frpc \
   --restart always \
   -e TZ=Asia/Shanghai \
   -v ./frpc:/opt/frpc \
   -d fatedier/frpc:v0.61.2 -c /opt/frpc/frpc.toml
```

同之前一样，日志文件是空白的，只要容器保持运行没有中途停止就算运行成功了

### 内网穿透SSH

先写一个 SSH 内网穿透的配置，将本机的 22 端口映射到远程的 8001 端口，编辑客户端配置文件`frpc.toml`在原基础上添加内容

```toml
# 与服务端建立连接，跟上面的配置要对应
serverAddr = "服务端IP地址"
serverPort = 7000

# 配置Token鉴权，要与服务端一致
auth.method = "token"
auth.token = "gbfvzhsybvtybsibvuipqfnnvlkashfgiawug"

# 配置日志信息
log.level = "warn"
log.to = "/opt/frpc/frpc.log"

# 该内网穿透起名为SSH，annotations中随便写了一些备注，基于TCP协议将本机的22端口映射到公网的8001
[[proxies]]
name = "SSH"
annotations = {title = "SSH远程连接", fuck = "test", desc = "annotations是该连接的备注信息，里面的key和val是随便写的，在服务端控制面板可以看到"}
type = "tcp"
localIP = "192.168.137.10"
localPort = 22
remotePort = 8001
```

内网穿透配置完成后`docker restart frpc`重启容器，就可以使用公网IP在外地SSH远程本机了，控制面板看到连接信息

### 内网穿透HTTP

FRP 支持 HTTP/HTTPS 协议的内网穿透，但是使用 HTTP 类型的内网穿透不是很方便，还需要配置一个域名，HTTPS 则更麻烦一些，还需要配置 SSL 证书，这里选择继续使用基于 TCP 协议的网站内网穿透

我本机运行了 Nginx，就以他为例子继续编辑客户端配置文件`frpc.toml`，在原基础上再在加一个内网穿透

```toml
# 与服务端建立连接，跟上面的配置要对应
serverAddr = "服务端IP地址"
serverPort = 7000

# 配置Token鉴权，要与服务端一致
auth.method = "token"
auth.token = "gbfvzhsybvtybsibvuipqfnnvlkashfgiawug"

# 配置日志信息
log.level = "warn"
log.to = "/opt/frpc/frpc.log"

# 该内网穿透起名为SSH，annotations中随便写了一些备注，基于TCP协议将本机的22端口映射到公网的8001
[[proxies]]
name = "SSH"
annotations = {title = "SSH远程连接", fuck = "test", desc = "annotations是该连接的备注信息，里面的key和val是随便写的，在服务端控制面板可以看到"}
type = "tcp"
localIP = "192.168.137.10"
localPort = 22
remotePort = 8001

# 该内网穿透起名为NGINX-HOME，基于TCP协议将本机的80端口映射到公网的8002
[[proxies]]
name = "NGINX-HOME"
type = "tcp"
localIP = "192.168.1.183"
localPort = 80
remotePort = 8002
```

`docker restart frpc`重启容器后访问公网查看效果