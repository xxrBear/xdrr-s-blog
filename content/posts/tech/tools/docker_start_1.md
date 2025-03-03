---
title: "Docker 入门指南"
date: 2025-03-03T20:35:05+08:00
lastmod: 2025-03-03T20:35:05+08:00
author: ["熊大如如"]
tags: # 标签
  - "docker"
description: ""
weight:
slug: ""
summary: "docker 入门，介绍 image 和 container"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image/202503032053243.png" # 文章的图片
---

## 一、Docker 是什么

### 1.1 简介

`Docker` 是一种开源的容器化平台，用于开发、部署和运行应用程序。它通过容器技术将应用程序及其依赖项打包在一起，确保在不同环境中一致运行。

核心是对 `Linux` 容器的一种封装，提供了一个简单易用的容器管理接口。

### 1.2 Docker 解决了什么问题

Docker 通过容器化技术解决了环境一致性、依赖冲突、部署复杂、资源利用率低等问题，提升了开发、测试和部署的效率，支持微服务架构和 CI/CD 流程，是现代软件开发和运维的重要工具。

## 二、安装

`Docker` 分为社区版和企业版，个人用户安装社区版即可。

### 2.1 Ubuntu

```shell
# 运行以下命令卸载旧版 Docker
sudo apt-get remove docker docker-engine docker.io containerd runc

# 更新 apt 包索引
sudo apt-get update

# 安装 apt 依赖包
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# 添加 Docker 的官方 GPG 密钥：
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# 安装 docker 社区版
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 添加docker用户组，避免每次都要输入 sudo
sudo usermod -aG docker $USER
```

### 2.2 Windows

在 `Windows` 上安装`Docker` 需要 `Hyper-V` 或 `WSL2`，有需求安装可自行搜索 🙂

## 三、基本概念

### 3.1 Image

`Docker` 镜像是一个只读的模板，包含了运行应用程序所需的所有内容，例如代码、运行时环境、库、环境变量和配置文件等。镜像是容器的基础，容器是镜像的运行实例。

### 3.2 Container

`Docker` 容器是一个轻量级、可移植的软件单元，用于打包和运行应用程序及其依赖项。它基于镜像创建，提供了隔离的运行环境，确保应用程序在不同环境中一致运行。容器具有高效、隔离、可移植等优点，广泛应用于应用部署、微服务、CI/CD 等场景。

## 四、Image

### 4.1 image 的构建方式

基本上，获取镜像的方式有以下几种：

1. 从 `Docker Hub` 拉取，`Docker Hub` 是 `Docker` 官方的公共镜像仓库，提供了大量官方和社区维护的 `Docker` 镜像。缺点国内访问速度慢，甚至无法访问
2. 自己编写`Dockerfile`构建镜像
3. 根据本地运行的容器，构建一个镜像

### 4.2 拉取镜像

```shell
docker image pull nginx
```

从 `Docker Hub` 中拉取 `nginx`镜像

~~不成功~~~~😂~~

### 构建镜像

下面单独聊

### 4.3 导出镜像

```shell
docker commit abc123def456 my_new_image:1.0
```

将 `abc123def456` 容器打包为一个名为 `my_new_image` 且标签为 `1.0` 的新镜像

## 五、Dockerfile 构建镜像

### 5.1 来个例子

创建一个 FastAPI 项目

```python
from fastapi import FastAPI

app = FastAPI()
```

安装 `fastapi`，并写入 `requirements.txt` 文件中

```shell
pip3 install fastapi

pip freeze > requirements.txt
```

让我们启动它

```shell
uvicorn main:app --reload
```

现在，让我们根据以上命令，写一个 Dockerfile 文件

```dockerfile
# 使用 Python 3.11 镜像作为基础镜像
FROM  docker.1ms.run/library/python:3.11-slim

# 设置工作目录
WORKDIR /app

# 将当前目录的内容复制到容器中的工作目录
COPY . /app

# 安装项目依赖 (如果有 requirements.txt 文件)
RUN pip install --no-cache-dir -r requirements.txt -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple

# 暴露容器的 8000 端口 (根据需求修改)
EXPOSE 8000

# 默认启动命令
CMD ["sh", "-c", "python init_db.py && uvicorn main:app --host 0.0.0.0 --port 8102"]
```

让我们逐一解释一下这些命令：

`FROM`：指定基础镜像

`WORKDIR`：会为后续的命令设置一个默认的工作目录

`COPY`：可以将主机上的文件或目录复制到容器内的指定路径

`RUN`：用于在镜像构建过程中执行命令，每个 `RUN` 指令都会创建一个新的镜像层。为了减少镜像层数和镜像大小，建议将多个命令合并到一个 `RUN` 指令中。

`EXPOSE`：用于声明容器运行时监听的网络端口，它不会自动将端口映射到宿主机，实际映射需要在运行容器时通过 `-p` 或 `-P` 参数指定。

`CMD`：用于指定容器启动时的默认执行命令

也就是说，上面的的`Dockerfile`文件的步骤是：

1. 拉取 python3.11 的镜像
2. 创建 `/app`作为工作目录
3. 复制当前文件夹的所有文件到 `/app`中
4. 运行 pip 下载依赖库
5. 暴露 8000 端口
6. 设置容器运行时的命令

> 注意：这里我们拉取镜像，使用的是第三方镜像地址

### 5.2 构建镜像

现在我们已经有了 Dockerfile 了，让我们来依据它来构建一个 Docker Image 吧，构建只需要一行命令

```shell
docker build -t my-fastapi .
```

这样，我们就构建成功了一个名字为 `my-fastapi`的镜像

## 六、Container

### 6.1 运行新容器

上面我们已经创建了一个名为`my-fastapi`的镜像，现在基于它生成新容器

```shell
docker run -d -p 8000:8000 my-fastapi
```

让我们解释一下这些参数

`docker run`：运行一个新的`Docker`容器

`-d detached mode`：以后台运行模式启动容器，使其在后台运行，而不会占用当前终端

`-p 8000:8000`：端口映射，将宿主机（本机）的 `8000` 端口映射到容器的 `8000` 端口

### 6.2 查看容器

让我们查看运行中的容器

```shell
docker container ls
```

你会看到如下项

```shell
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

让我们逐一解释：

**CONTAINER ID**

含义: 容器的唯一标识符（ID）

说明:

- 这是一个由 Docker 自动生成的哈希值，用于唯一标识一个容器
- 通常只显示前 12 个字符，但可以通过 `docker inspect` 查看完整的 ID

示例: `a1b2c3d4e5f6`

---

**IMAGE**

含义: 容器所使用的镜像名称

说明:

- 镜像是容器的基础，容器是从镜像创建的实例
- 如果镜像有标签（如 `nginx:latest`），标签也会显示在这里

示例: `nginx`, `ubuntu:20.04`

---

**COMMAND**

含义: 容器启动时执行的命令

说明:

- 这是容器启动时运行的默认命令或用户指定的命令
- 如果命令较长，可能会被截断显示

示例: `/bin/bash`, `nginx -g 'daemon off;'`

---

**CREATED**

含义: 容器的创建时间

说明:

- 显示容器从创建到当前时间的时间差（如 `2 hours ago`）
- 可以通过 `docker inspect` 查看具体的创建时间

示例: `2 hours ago`, `5 minutes ago`

---

**STATUS**

含义: 容器的当前状态。

说明:

- 常见的状态包括：
  - `Up`：容器正在运行
  - `Exited`：容器已停止
  - `Restarting`：容器正在重启
  - `Paused`：容器已暂停
- 状态后面通常会附带时间信息（如 `Up 2 hours` 或 `Exited (0) 5 minutes ago`）

---

**PORTS**

含义: 容器的端口映射信息

说明:

- 显示容器内部端口与主机端口的映射关系
- 如果没有端口映射，则显示为空
- 格式为 `<主机端口>-><容器端口>/<协议>`（如 `8080->80/tcp`）

示例: `0.0.0.0:8080->80/tcp`, `(无)`

---

**NAMES**

含义: 容器的名称。

说明:

- 容器的名称可以由用户通过 `--name` 参数指定
- 如果没有指定名称，Docker 会随机生成一个名称
- 名称是容器的唯一标识符之一，可以在命令中代替容器 ID 使用

示例: `my-nginx`, `happy_mendeleev`

## 七、常用命令

### 7.1 image 常用命令

```shell
# 列出本地所有的镜像
docker images

# 删除本地镜像
docker rmi <image_id_or_name>

# 强制删除镜像
docker rmi -f <image_id_or_name>

# 删除所有未使用的镜像：
docker image prune -a

# 14. 重命名镜像
docker tag my-app:1.0 my-new-app:1.0
```

### 7.2 container 常用命令

```shell
# 指定运行容器名称
docker run --name my-container <image_name>

# 查看正在运行的容器
docker ps
docker container ls

# 查看所有容器
docker ps -a
docker container ls -a

# 启动/停止/重启容器
docker start <container_id_or_name>
docker stop <container_id_or_name>
docker restart <container_id_or_name>

# 删除已停止的容器
docker rm <container_id_or_name>

# 强制删除运行中的容器
docker rm -f <container_id_or_name>

# 删除所有已停止的容器
docker container prune

# 在运行中的容器中执行命令
docker exec -it my-nginx /bin/bash

# 查看容器的日志输出
docker logs -f <container_id_or_name>
```

### 7.3 Dockerfile 常用指令

| 指令          | 作用                               |
| ------------- | ---------------------------------- |
| `FROM`        | 指定基础镜像                       |
| `RUN`         | 在构建时执行命令                   |
| `CMD`         | 容器启动时默认执行的命令           |
| `ENTRYPOINT`  | 容器启动时执行的主命令             |
| `WORKDIR`     | 设置容器内的工作目录               |
| `COPY`        | 复制本地文件到镜像                 |
| `ADD`         | 复制本地文件或远程文件，并自动解压 |
| `EXPOSE`      | 声明容器的对外端口                 |
| `ENV`         | 设置环境变量                       |
| `ARG`         | 定义构建参数                       |
| `VOLUME`      | 定义挂载点                         |
| `LABEL`       | 给镜像添加元数据                   |
| `USER`        | 指定运行容器的用户                 |
| `HEALTHCHECK` | 定义健康检查命令                   |

## 八、参考链接

- [Docker 入门教程](https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
- [目前国内可用 Docker 镜像源汇总](https://www.coderjia.cn/archives/dba3f94c-a021-468a-8ac6-e840f85867ea)
