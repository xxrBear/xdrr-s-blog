---
title: "Github Pages 自定义域名访问"
date: 2024-08-10T20:59:24+08:00
lastmod: 2024-08-10T20:59:24+08:00
author: ["熊大如如"]
tags: # 标签
 - ["github"]
summary: "利用 github pages 搭建一个个人站点，并且自定义域名访问"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---

最近突发奇想，想用 GitHub Pages 搭建一个自己的知识体系小网站，又不想使用 github 自带的子域名，于是折腾了一下自定义域名访问 GitHub Pages 

记录如下

### 一、购买域名
国内购买域名都要备案，少则一两周，多则一个月，审查太麻烦不方便，买国外的。

虽然国外买域名的网站很多但是支持国内支付方式的不多，我选择 NameSilo, 支持支付宝支付

[NameSilo](https://www.namesilo.com/)

搜索你想要购买的域名

上面的是购买价格，下面的是续费价格

![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202408102153669.png)

### 二、设置 Cloudflare 托管
Cloudflare可以为我们设置 CDN 服务、DNS 域名解析和一些防止 DDOS 攻击的服务

点击右上角 添加站点

选择 free 服务
![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202408102245329.png)

将以下网址写入你的域名网站的解析上

    sid.ns.cloudflare.com
    sneh.ns.cloudflare.com

点击 **Domain Manager**

![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202408102247446.png)
![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202408102252709.png)
![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202408102253705.png)
稍等一段时间，等待 Cloudflare 解析成功，界面如下就是成功了
![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202408102253071.png)

### 三、设置 Github 域名配置
设置 GitHub 的域名如下，这样你就可以通过自定义域名访问 Github Pages了
![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202408102254493.png)
