---
title: "Tmux 快速入门"
date: 2024-09-12T16:07:26+08:00
lastmod: 2024-09-12T16:07:26+08:00
author: ["熊大如如"]
tags: # 标签
  - ["tmux", "Linux"]
summary: "tmux 快速入门指南"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202411102031289.png"
---

## 一、Tmux 是什么

tmux 意为(terminal multiplexer)， 是一个终端复用软件。

## 二、快速安装

### 2.1 Mac

    brew install tmux

## 三、会话管理

### 3.1 新建会话

    tmux new -s 0 ｜ <name>

### 3.2 分离会话

    tmux detach

### 3.3 接入会话

    tmux attach - 0 ｜ <name>

### 3.4 杀死会话

    tmux kill-session -t 0

### 3.5 切换会话

    tmux switch -t 0

### 3.6 会话快捷键
- Ctrl+b d：分离当前会话。
- Ctrl+b s：列出所有会话。
- Ctrl+b $：重命名当前会话。

## 四、窗口操作

### 4.1 新建窗口

    tmux new-window

### 4.2 重命名窗口

    tmux rename-window <new-name>

## 五、参考链接
- [阮一峰 Tmux 使用教程](https://www.ruanyifeng.com/blog/2019/10/tmux.html)