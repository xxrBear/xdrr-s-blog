---
title: "Hugo博客自动部署实践：GitHub Actions入门实战"
date: 2025-02-09T15:21:52+08:00
lastmod: 2025-02-09T15:21:52+08:00
author: ["熊大如如"]
tags: # 标签
  - "Hugo"
description: ""
weight:
slug: ""
summary: "Hugo博客自动部署实践"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502091523060.png" # 文章的图片
---

自前年开始，我尝试使用 Hugo 搭建自己的博客系统，用以记录工作生活和技术笔记。最近闲来无事，利用 GitHub Actions 将原本手动部署博客的流程改为自动化部署，现将过程整理如下，与大家分享。

## 一、原有的博客部署流程

1.在 Hugo 工程中写博客文章

2.在本地运行 Hugo 预览，检查页面样式和内容

3.确认无误后，将代码提交推送至 GitHub 仓库

4.登录腾讯云服务器，手动拉取 GitHub 上的代码

5.在服务器上使用 Hugo 生成所有静态博客页面

## 二、为何更换原流程

1.每次都要在服务器上手动拉取代码，太麻烦，而且腾讯云拉 GitHub 代码可能会很慢，还需要配置 host 文件

2.借助 GitHub Actions 能实现自动化部署，同时还能让我学习和实践新的 CI/CD 技术 :)

## 三、GitHub Actions 是什么

GitHub Actions 是 GitHub 平台内置的 CI/CD（持续集成/持续部署）自动化工具。它允许你通过 YAML 文件定义各种工作流，在代码推送、拉取请求等事件触发时自动运行，实现自动构建、测试、打包及部署。利用这一工具，可以大幅度减少重复性人工操作，让开发者专注于高价值工作

## 四、自动化部署流程设计

原部署流程的 1-3 步还是需要手动完成的，自动化的部分则涵盖步骤 4-5，整体思路为：利用 GitHub Actions 自动编译 Hugo 博客，并将生成的 public 文件夹通过 scp 命令上传到腾讯云服务器，借助配置好的 Nginx 代理展示静态博客。

**为什么不使用其他的自动化流程呢？**

因为这已经能满足我的需求了，重要的是别的流程我看不明白 😅

## 五、实现步骤

### 5.1 创建工作流配置文件

在项目根目录下创建 `.github/workflows/deploy.yml` 文件

### 5.2 配置仓库 secrets

在 **GitHub** 仓库的 **Settings → Secrets and variables → Actions** 中，添加以下 **Secrets**：

- `SERVER_IP`: 腾讯云 IP
- `SERVER_USER`：腾讯云用户名
- `SERVER_PASSWORD`：腾讯云密码

### 5.3 文件内容

```yaml
name: Deploy My Blog to Tencent Cloud Lighthouse 🚀

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-22.04

    steps:
      - name: 检出代码
        uses: actions/checkout@v3
        with:
          submodules: true # 启用 Git 子模块

      - name: 更新 Git Submodules 并切换到指定 Tag
        run: |
          git submodule update --init --recursive --remote
          cd themes/hugo-PaperMod
          git fetch --tags
          git checkout tags/v7.0
          cd ../..

      - name: 安装 Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "0.119.0" # 指定 Hugo 版本
          extended: true # 开启 Hugo Extended 版（支持 SCSS/SASS）

      - name: 生成静态文件
        run: hugo -F --cleanDestinationDir

      - name: 通过 SSH 发送文件
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ${{ secrets.SERVER_USER }}
          password: ${{ secrets.SERVER_PASSWORD }}
          # 指定上传 Hugo 生成的 public 文件夹中的所有内容
          source: "public/*"
          # 指定腾讯云服务器上目标目录
          target: "/home/ubuntu/myblog/flow/"
```

- 我的 Hugo 博客主题是 hugo-PaperMod，自动化构建时需要指定子模块
- 使用的 Hugo 版本是 0.121.1 本地没有问题，但是自动化生成的内容有样式问题，在查看 [Issues-1325](https://github.com/adityatelange/hugo-PaperMod/issues/1325) 后，通过降级版本解决。

## 总结

通过上面的流程，我们将手动部署博客的过程转变为自动化部署，不仅节省了人工拉取代码和生成静态文件的时间，还使整个部署过程更加稳定高效。希望这篇记录对有相同需求的朋友有所帮助，也期待在实践中不断优化和学习新的自动化工具。

这就是我的博客自动化部署实践的详细流程，不足之处，烦请指正~
