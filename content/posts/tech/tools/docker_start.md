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
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202411102036348.png" # 文章的图片
---

## 一、Docker 出现前的世界

在软件开发过程中，环境搭建是最繁琐的事情之一。不同用户的环境可能存在差异，导致一个软件在开发者的机器上运行良好，却在他人的设备上无法正常运行。

## 二、虚拟机

使用 Virtualbox 等工具运行 Linux 操作系统有许多缺点。例如

- 配置步骤复杂且耗时
- 占用宿主机大量内存和硬盘空间
- 启动速度慢

## 三、Linux 容器

linux 容器是对进程的封装，在进程外面套上一层“保护层”，具有以下优点：

- 进程级别，启动快
- 占用资源少
- 体积小

## 四、Docker 是什么

Docker 是对 Linux 容器的一种封装，提供了一个简单易用的容器管理接口。

虽然 Docker 的实现较为复杂，但其原理相对简单：通过最小化的 Linux 环境，保证应用的独立运行。

## 五、Docker 的安装

Docker 分为社区版和企业版，个人用户安装社区版即可。

**Windows / Mac 用户**

[官网下载 Desktop](https://www.docker.com/)

**Linux 用户(Ubuntu 为例)**

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

# 添加docker用户组
sudo usermod -aG docker $USER
```

## 六、参考链接

- [Docker 入门教程（阮一峰）](https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
- [菜鸟教程](https://www.runoob.com/docker/ubuntu-docker-install.html)
