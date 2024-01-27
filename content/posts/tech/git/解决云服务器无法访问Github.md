---
title: "解决云服务器无法拉取GitHub代码"
date: 2024-01-27T12:36:28+08:00
lastmod: 2024-01-27T12:36:28+08:00
author: ["熊大如如"]
keywords: 
- 
tags: # 标签
    - "git"
description: ""
weight:
slug: ""
summary: "解决在腾讯云中无法访问GitHub的问题"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: false # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---

### 1.问题描述
```text
git pull origin master
```
在腾讯云服务器上拉取GitHub上的代码时发现无法拉取


### 2.检查自己的云服务器是否能访问GitHub
```
ping github.com
```
> 发现访问不通

### 3.修改hosts文件
```
sudo vim /etc/hosts
```
### 4.添加以下内容
```text
140.82.114.25                 alive.github.com
140.82.112.25                 live.github.com
185.199.108.154               github.githubassets.com
140.82.112.22                 central.github.com
185.199.108.133               desktop.githubusercontent.com
185.199.108.153               assets-cdn.github.com
185.199.108.133               camo.githubusercontent.com
185.199.108.133               github.map.fastly.net
199.232.69.194                github.global.ssl.fastly.net
140.82.112.4                  gist.github.com
185.199.108.153               github.io
140.82.114.4                  github.com
192.0.66.2                    github.blog
140.82.112.6                  api.github.com
185.199.108.133               raw.githubusercontent.com
185.199.108.133               user-images.githubusercontent.com
185.199.108.133               favicons.githubusercontent.com
185.199.108.133               avatars5.githubusercontent.com
185.199.108.133               avatars4.githubusercontent.com
185.199.108.133               avatars3.githubusercontent.com
185.199.108.133               avatars2.githubusercontent.com
185.199.108.133               avatars1.githubusercontent.com
185.199.108.133               avatars0.githubusercontent.com
185.199.108.133               avatars.githubusercontent.com
140.82.112.10                 codeload.github.com
52.217.223.17                 github-cloud.s3.amazonaws.com
52.217.199.41                 github-com.s3.amazonaws.com
52.217.93.164                 github-production-release-asset-2e65be.s3.amazonaws.com
52.217.174.129                github-production-user-asset-6210df.s3.amazonaws.com
52.217.129.153                github-production-repository-file-5c1aeb.s3.amazonaws.com
185.199.108.153               githubstatus.com
64.71.144.202                 github.community
23.100.27.125                 github.dev
185.199.108.133               media.githubusercontent.com
```

### 5.尝试ping GitHub网站
```text
ping github.com
PING github.com (140.82.114.4) 56(84) bytes of data.
64 bytes from github.com (140.82.114.4): icmp_seq=1 ttl=45 time=188 ms
64 bytes from github.com (140.82.114.4): icmp_seq=2 ttl=45 time=188 ms
```
**可以ping通**

### 6.再次拉取代码
```text
git pull origin master
From https://github.com/xxrBear/blog
 * branch            master     -> FETCH_HEAD
Already up to date.

```
**成功拉到代码**