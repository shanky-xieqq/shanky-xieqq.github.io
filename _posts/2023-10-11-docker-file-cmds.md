---
layout: post
title:  "Dockerfile的语法与常用命令"
categories: Docker
tags: Docker
---

* content
{:toc}

记录一下Dockerfile的语法和命令





一、Dockerfile 核心语法规则
-------------------

1.  指令大写：所有指令必须大写（如 `FROM`, `RUN`）
2.  顺序执行：指令按顺序从上到下执行
3.  分层构建：每条指令生成一个镜像层，修改上层不会影响下层
4.  注释支持：使用 `#` 符号添加注释
5.  基础镜像：必须包含 `FROM` 指令作为第一条指令

二、常用指令详解
--------

| 指令 | 作用 | 示例 |
| --- | --- | --- |
| FROM | 指定基础镜像 | `FROM ubuntu:20.04` |
| RUN | 执行命令并提交结果 | `RUN apt-get update && apt-get install -y curl` |
| COPY | 复制本地文件到镜像 | `COPY ./app /app` |
| ADD | 复制文件并支持自动解压（推荐优先使用 COPY） | `ADD https://example.com/file.tar.gz /tmp` |
| CMD | 定义容器启动默认命令（可被覆盖） | `CMD ["python", "app.py"]` |
| ENTRYPOINT | 定义容器启动主命令（不可被覆盖，可组合使用） | `ENTRYPOINT ["python"]` |
| WORKDIR | 设置工作目录 | `WORKDIR /app` |
| EXPOSE | 声明容器监听端口 | `EXPOSE 80` |
| ENV | 设置环境变量 | `ENV NODE_ENV=production` |
| VOLUME | 创建挂载点 | `VOLUME /data` |
| ARG | 定义构建参数（仅在构建时有效） | `ARG USER_ID=1000` |
| USER | 指定运行用户 | `USER root` |
| LABEL | 添加元数据 | `LABEL maintainer="your@email.com"` |
| ONBUILD | 定义镜像作为其他镜像基础时执行的指令（较少使用） | `ONBUILD RUN echo "Building child image"` |

三、典型使用场景示例
----------

### 示例1：构建 Node.js 应用镜像

[![Image 13: 复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\# 使用官方 Node.js 镜像作为基础
FROM node:18\-alpine

# 设置工作目录
WORKDIR /app

# 复制 package.json 并安装依赖
COPY package\*.json ./
RUN npm install \--production

# 复制应用代码
COPY . .

# 暴露应用端口
EXPOSE 3000

# 定义启动命令
CMD \["npm", "start"\]

[![Image 14: 复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

构建命令：

docker build -t my-node-app .

运行命令：

docker run -p 3000:3000 my-node-app

### 示例2：Python Web 服务镜像

[![Image 15: 复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\# 使用 Python 官方镜像
FROM python:3.11\-slim

# 设置环境变量
ENV PYTHONUNBUFFERED\=1

# 创建工作目录
WORKDIR /code

# 安装系统依赖
RUN apt\-get update && apt-get install -y --no-install-recommends \\
    gcc \\
    python3\-dev \\
    && rm -rf /var/lib/apt/lists/\*

# 复制依赖文件并安装
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 启动命令
CMD \["gunicorn", "--bind", "0.0.0.0:8000", "myapp.wsgi"\]

[![Image 16: 复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

2.构建优化技巧：

1.  使用 `.dockerignore` 文件排除无关文件
2.  合并 RUN 指令减少层数：

RUN apt-get update \\
    && apt-get install -y package \\
    && rm -rf /var/lib/apt/lists/\*

3.多阶段构建（减小最终镜像体积）：

[![Image 17: 复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

\# 构建阶段
FROM golang:1.21 as builder
WORKDIR /app
COPY . .
RUN go build \-o myapp

# 最终阶段
FROM alpine:latest
COPY \--from\=builder /app/myapp /app/
CMD \["/app/myapp"\]

[![Image 18: 复制代码](https://assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

四、关键注意事项
--------

1.  层缓存机制：Dockerfile 修改后，只有修改后的指令及其后续指令会重新执行
2.  安全最佳实践：
    *   避免以 root 用户运行最终容器
    *   使用最小基础镜像（如 alpine）
    *   定期更新基础镜像
3.  多阶段构建：适用于需要编译环境但运行时不需要的情况（如 Go/C++ 应用）
4.  健康检查：
    
    HEALTHCHECK --interval=30s --timeout=3s \\
      CMD curl \-f http://localhost/ || exit 1
    

五、调试技巧
------

1.  进入容器调试：
    
    docker run -it my-image /bin/bash
    
2.  查看构建历史：
    
    docker history my-image
    
3.  查看镜像详细信息：
    
    docker inspect my-image