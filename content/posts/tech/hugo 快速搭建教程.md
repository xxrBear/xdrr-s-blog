---
title: "hugo 第一篇测试文章"
date: 2023-12-11T14:28:07+08:00
lastmod: 2023-12-11T14:28:07+08:00
author: [ "熊大如如" ]
keywords:
  -
categories: # 没有分类界面可以不填写
  -
tags: # 标签
  - "hugo"
description:
  - "hugo 第一篇测试文章"
weight:
slug: "hugo 第一篇测试文章"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: false # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
subtitle: "描述"
---


## 1.hugo是什么
hugo是一个go语言开发的，静态网站生成器。


## 2.如何下载hugo
- Windows
```
winget install Hugo.Hugo.Extended
```
- MacOS
```
brew install hugo
```
- Linux
```
sudo apt install hugo
```

## 3.创建一个新的hugo项目
```
hugo new site myblog
```

## 4.启动hugo项目
- hugo:会在项目文件夹下生成public文件
- hugo server:启动hugo自带的开发者服务器，可以用来预览效果
- hugo -F --cleanDestinationDir:生成一个全新的public文件夹

### 更多参考
- [Sulv's Blog](https://www.sulvblog.cn)
- [Hugo官方文档](https://gohugo.io/documentation/)
