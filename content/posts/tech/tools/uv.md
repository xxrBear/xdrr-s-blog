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

一个用 Rust 写的 Python 包和项目管理器

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

### 管理 Python

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

### 依赖管理

```shell
# 添加依赖
uv add requests

# 移除依赖
uv remove requests

# 同步依赖
uv sync

# 安装依赖：如果 requirements.txt 或 pyproject.toml 中定义了新依赖，uv sync 会安装它们
# 卸载多余依赖：如果当前环境中存在未在 requirements.txt 或 pyproject.toml 中定义的依赖，
# uv sync 会自动删除它们，以保持环境的干净
```

### 运行命令

`uv run` 命令的作用是在 uv 管理的虚拟环境中运行命令

`uv run` 的主要功能

1. 自动激活虚拟环境：在 `uv venv` 创建的环境中执行命令，而不需要手动 `source venv/bin/activate`
2. 执行 Python 脚本：可以直接运行 Python 相关命令，如 `python`、`pytest`、`flask run` 等
3. 运行任意终端命令：不仅限于 Python，还可以运行 `bash`、`sh` 等

> 即，uv run 的作用是激活当前 uv 项目的虚拟环境

### 运行二进制文件

`uvx` 的作用是在 UV 虚拟环境中运行可执行文件，相当于 `uv run`，但专门用于运行可执行二进制文件

```shell
uvx black .

uvx ruff check .

uvx mypy my_script.py
```

什么时候用 `uvx`？

- 运行已安装的 CLI 工具（`black`、`ruff`、`mypy`、`pyright`）
- 确保使用 UV 虚拟环境中的二进制文件，而不是系统全局版本
- 在 CI/CD 或 Docker 中执行格式化、静态检查等任务

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
