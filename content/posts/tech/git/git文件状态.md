---
title: "Git 文件状态"
date: 2025-02-16T12:51:25+08:00
lastmod: 2025-02-16T12:51:25+08:00
author: ["熊大如如"]
tags: # 标签
  - "git"
description: ""
weight:
slug: ""
summary: "git 文件状态的流转"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image/git-image.png"
---

## Git 的故事

Git 是 Linus 为了管理 linux 源码而开发的分布式版本控制工具

原谅我如此简单的介绍 Git，如果你对 Git 诞生的经历有兴趣，我推荐你观看这篇文章

[Git 的故事：這一次沒這麼好玩](https://blog.brachiosoft.com/posts/git/)

文章是繁体中文的，详细介绍了 Git 的故事

## Git 的文件状态

对于 Git 仓库中的文件，我们可以将他们分为 ` 已追踪(Tracked)`和 `未追踪(Untracked)`两部分

对于 `已追踪`的文件，可以分为三个状态：`已提交(committed)`、`已修改(modified)` 和 `已暂存(staged)`

## Git 文件状态的流转

一图胜千言

![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502161252206.png)

如有不足，欢迎指正~
