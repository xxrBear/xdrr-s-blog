---
title: "PicGo配置GitHub图床和CDN加速"
date: 2024-01-13T20:48:28+08:00
lastmod: 2024-01-13T20:48:28+08:00
author: ["熊大如如"]
tags: # 标签
  - "GitHub"
summary: "PicGo配置GitHub图床和CDN加速的步骤"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---

### 1.创建一个新的公共的仓库
[点击此处直达](https://github.com/new)
![](https://cdn.jsdelivr.net/gh/xxrBear/image//202401281306403.png)

### 2.生成一个token让PicGo操作你的仓库
[点击此处直达](https://github.com/settings/tokens)
+ 点击 Generate new token
+ Expiration 设置 No expiration
+ repo 勾选

### 3.配置您的PicGo的GitHub设置
![](https://cdn.jsdelivr.net/gh/xxrBear/image/202401281302853.png)

### 4.设置CDN加速
这样确实可以将图片上传到GitHub，但是我们在国内访问还是很难加载出来。

设置如下配置在自定义域名上

`https://cdn.jsdelivr.net/gh/你的GitHub昵称/你的GitHub图床仓库名/`


