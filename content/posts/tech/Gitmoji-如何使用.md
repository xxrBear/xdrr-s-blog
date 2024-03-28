---
title: "Gitmoji 如何使用"
date: 2024-03-28T14:19:40+08:00
lastmod: 2024-03-28T14:19:40+08:00
author: ["熊大如如"]
keywords: 
- 
categories: # 分类
- # 在这儿写分类
tags: # 标签
  - "git"
summary: "gitmoji 一个给git提交加点表情的工具"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cloud.githubusercontent.com/assets/7629661/20073135/4e3db2c2-a52b-11e6-85e1-661a8212045a.gif"  # 文章的图片
---

### 1.gitmoji是什么
当你像git提交信息时，是不是感觉空荡荡的文字有点儿无聊。gitmoji就是为你的提交信息增加点表情的工具。

### 2.下载gitmoji
- [下载最新版nodejs](https://nodejs.org/en)
- 下载gitmoji
    ```shell
    npm i -g gitmoji-cli
    ```

### 3.提交信息
```shell
git commit -m ':tada: 测试代码'
```

显示如下

🎉 测试代码
