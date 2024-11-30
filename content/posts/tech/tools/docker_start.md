---
title: "Docker 入门教程"
date: 2024-11-10T20:17:27+08:00
lastmod: 2024-11-10T20:17:27+08:00
author: ["熊大如如"]
tags: # 标签
    - "docker"
description: ""
weight:
slug: ""
summary: "docker系列入门教程，简单的介绍下docker"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: false # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202411102036348.png"  # 文章的图片
---

## 一、Docker 出现前的世界
在软件开发过程中，环境搭建是最繁琐的事情之一（另外两个是缓存失效和变量命名）。不同用户的环境可能存在差异，导致一个软件在开发者的机器上运行良好，却在他人的设备上无法正常运行。

在 Docker 出现之前，解决环境兼容问题的主要方式是使用虚拟机。例如，在 Windows 上运行 Linux 虚拟机。然而，启动虚拟机不仅速度较慢，还占用大量内存和硬盘空间，配置步骤复杂且耗时。这些缺点使虚拟机成为了不够理想的解决方案。

## 二、Docker 是什么
为了解决以上问题，Docker 应运而生。Docker 是对 Linux 容器的一种封装，提供了一个简单易用的容器管理接口。

虽然 Docker 的实现较为复杂，但其原理相对简单：通过最小化的 Linux 环境，保证应用的独立运行。

## 三、Docker 的安装
Docker 分为社区版和企业版，个人用户安装社区版即可。

- Windows / Mac 用户

    前往 Docker 官网下载 [Docker Desktop](https://www.docker.com/)

- Ubuntu 用户

    运行以下命令卸载旧版 Docker：

```shell
apt-get remove docker docker-engine docker.io containerd runc
```

## 四、Docker 的简单使用


以下是 Docker 的一些简单操作：

```shell
# 拉取 Nginx 镜像
docker pull nginx

# 运行 Nginx 镜像
docker container run nginx