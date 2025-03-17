---
layout: post
title:  "CentOS8解决Nginx的PermissionDenied问题"
categories: Nginx
tags: Centos Nginx
---

* content
{:toc}

在 CentOS 8 操作系统上安装配置 Nginx 时，遇到了 `Permission Denied` 的问题。尝试使用 `chown` 和 `chmod` 进行权限调整无果后，定位到是 SELinux 导致的问题。







## 解决方案

### 1. 安装和启动 Nginx

首先，确保正确安装并启动 Nginx：

```bash
sudo yum install -y nginx
systemctl start nginx.service
```

默认配置文件位于 `/etc/nginx/nginx.conf`，自定义配置通常放在 `conf.d` 目录下，例如              
`$nginx_conf/conf.d/default.conf。`             

添加一个简单的自定义项目配置：          

```bash
server {
    listen 8081;
    server_name localhost;
    access_log /var/log/nginx/access.log main;

    location / {
        root /home/custom/web; # 自定义路径
        index index.html index.htm;
    }
}
```

### 2. 权限问题

启动 Nginx 后，访问页面出现 403 错误（权限被拒绝），查看日志信息如下：

```bash
2018/09/18 23:41:37 [error] 1266#1266: *1 "/home/custom/web/index.html" is forbidden (13: Permission denied), client: xxx.xxx.xxx.xxx, server: localhost, request: "GET / HTTP/1.1", xxx.xxx.xxx.xxx:8081"
```

* 方法1: 使用 root 用户启动

一种解决方案是修改 nginx.conf 文件，将用户设置为 root：

```bash
user root;
worker_processes 1;
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    ...
}
```

但这种方法存在严重的安全隐患，不建议在生产环境中使用。

* 方法2: 配置 SELinux

通过 SELinux 设置正确的安全上下文来解决问题：

检查默认配置目录的安全上下文：

```bash
ll -Zd /usr/share/nginx/html/
# drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /usr/share/nginx/html/
```

对新的目录应用相同的上下文：

```bash
chcon -Ru system_u /home/custom/web
chcon -Rt httpd_sys_content_t /home/custom/web
```

然后，关闭 SELinux 或者将 httpd_t 类型设为宽容模式：

```bash
setenforce 0
# 或者
semanage permissive -a httpd_t
```

### 3. SELinux 日志分析

可以通过以下命令查看 Nginx 依赖的安全信息：

```bash
grep nginx /var/log/audit/audit.log | audit2allow -m nginx
```

### 4. 错误处理

当开启 SELinux 并尝试启动 Nginx 时，可能会遇到类似如下的错误日志：

```bash
[root@localhost mgzy]# systemctl start nginx
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
```

进一步查看日志发现：

```bash
nginx: [emerg] open() "/etc/nginx/none" failed (13: Permission denied)
```

此时，需要执行以下命令使 httpd_t 类型进入宽容模式：

```bash
semanage permissive -a httpd_t
```

至此，在 SELinux 开启的情况下，Nginx 可以正常工作。