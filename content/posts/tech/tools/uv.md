---
title: "uv 入门指南"
date: 2025-03-01T15:55:30+08:00
lastmod: 2025-03-01T15:55:30+08:00
author: ["熊大如如"]
keywords:
  -
categories: # 分类
  -  # 在这儿写分类
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "uv 一个 Rust 写的 Python 包管理器"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" # 文章的图片
---

## 简介

一个用 Rust 写的 Python 包和项目管理器。

先看它自己怎么吹的~

- 🚀 一款工具可替代 `pip` `pip-tools` `pipx` `poetry` `pyenv` `twine` `virtualenv` 以及更多。
- ⚡️ 比 `pip` 快 10-100 倍。
- 🗂️ 提供全面的项目管理，具有通用的锁文件。
- ❇️ 运行脚本，支持内联依赖元数据。
- 🛠️ 运行并安装作为 Python 包发布的工具。
- 🔩 包含与 pip 兼容的接口，以熟悉的命令行界面提升性能。
- 🖥️ 支持 macOS、Linux 和 Windows。

### 安装

```shell
# Linux & MacOS
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

## 基本使用

### 包管理

```shell
# 添加包
uv add requests

# 移除包
uv remove requests
```

### 管理不同版本 Python

```shell
# 寻找当前可用python解释器
uv python find

# 寻找可安装python解释器
uv python list

# 下载python解释器
uv python install 3.13

# 卸载 Python 版本
uv python uninstall

# 固定 Python 版本
uv python pin 3.13

# 安装虚拟环境，默认名字 .venv
uv venv --python 3.11
```

## 项目结构

```shell
.python-version
pyproject.toml
uv.lock
```

`.python-version`：uv 使用的 python 版本

`pyproject.toml`: uv 的元数据信息

`uv.lock`：用于确保所有依赖的版本一致，避免团队或生产环境中的版本差异问题。

### 修改 pip 源

```shell
# 修改 pip 源

# 添加到 pyproject.toml文件中
[[tool.uv.index]]
url = "https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple" # 清华源
```
