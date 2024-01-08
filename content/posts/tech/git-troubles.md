---
title: "Git Troubles"
date: 2024-01-04T19:56:09+08:00
lastmod: 2024-01-04T19:56:09+08:00
author: ["熊大如如"]
tags: # 标签
  - "git"
description: ""
weight:
slug: ""
summary: "git问题集合"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---

## 1.git解决中文乱码问题
```
git config --global core.quotepath false
```

## 2.Ubuntu安装最行版本git
```
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git
```