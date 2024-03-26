---
title: "Python 虚拟环境"
date: 2024-03-26T16:36:11+08:00
lastmod: 2024-03-26T16:36:11+08:00
author: ["熊大如如"]
keywords: 
- 
categories: # 分类
- # 在这儿写分类
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "python虚拟环境基础知识"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: ""  # 文章的图片
---

### 1.Python虚拟环境存在的意义
试想一下，你的公司有两个Django项目，一个是django1.11版本，一个是django4.2版本，而你本地只有一个python解释器，那你如何避免这两个不同版本的django之间的影响呢？

如果将两个django包下载到同一个python第三方包目录下肯定不行，python也不允许你这么做。

那么虚拟环境就完美的解决了这一问题，通过创建虚拟环境，你可以复制出两个相互隔绝的python解释器环境，避免了两个不同版本django包的干扰，它相当于复制了本地的python解释器环境到指定的地方，从而避免了这类问题。

### 2.Python虚拟环境的管理包工具
现在，管理Python虚拟环境的工具数不胜数，这里简单列出几个
- venv: [文档](https://docs.python.org/zh-cn/3/library/venv.html)
- virtualenv: [文档](https://virtualenv.pypa.io/en/latest/#)  
- pyenv: [文档](https://github.com/pyenv/pyenv?tab=readme-ov-file#how-it-works)   
- pipenv: [文档](https://pipenv.pypa.io/en/latest/)  

### 3.virtualenv工具管理虚拟环境
因为管理Python虚拟环境的工具包太多，每一个都学会太浪费时间(个人想法)，所以这里只介绍virtualenv，个人觉得简单易用。

- `下载virtualenv`
    ```python
    pip install virtualenv
    ```

- `创建虚拟环境`
    ```python
    virtualenv venv
    ```

- `激活虚拟环境`
    ```shell
    source venv/bin/activate
    ```

- `退出虚拟环境`
    ```shell
    deactivate
    ```