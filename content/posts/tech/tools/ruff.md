---
title: "Ruff 高性能Python代码检查工具"
date: 2025-03-03T12:03:46+08:00
lastmod: 2025-03-03T12:03:46+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Ruff 入门指南"
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

## Linter 是什么

Linter 是一种静态代码分析工具，用于检查源代码中的错误、潜在问题、风格违规以及不良实践。它的名字来源于最早的 C 语言静态分析工具 `lint`，后来逐渐演化为泛指所有语言的类似工具。

## 简介

Ruff 一个用 Rust 编写的极快的 Python linter 和代码格式化程序。

- ⚡️ 比现有的 linters（如 Flake8）和格式化程序（如 Black）快 10-100 倍
- 🐍 可通过以下方式安装`pip`
- 🛠️`pyproject.toml`支持
- 🤝 Python 3.13 兼容性
- 📦 内置缓存，避免重新分析未改变的文件
- 🔧 修复支持，用于自动更正错误（例如，自动删除未使用的导入）

## 安装

一般 ruff 会与 uv 配合使用

### 使用 uv 安装（推荐）

```shell
uv add --dev ruff
```

### 使用 pip 安装

```shell
pip3 install ruff
```

## 入门实践

### 代码格式化

让我们创建一个 main.py 文件

```shell
def hello():
  print('hello world!')
```

格式化代码

```shell
uv run ruff format main.py
```

查看结果

```shell
def hello():
    print("hello world!")
```

我们看到，缩进已经正确变成了四个空格，而不是两个，单引号也变成了双引号，但是我们并不想设置如此严格的规则，因为这会引起那些喜欢使用单引号的开发者的抗拒

让我们设置参数，使得引号保持原状

```shell
# 修改pyproject.toml 或 ruff.toml

[tool.ruff.format]
quote-style = "preserve"

or

[format]
quote-style = "preserve"
```

### 代码检查

`ruff check` 命令是 Ruff 工具中的一个子命令，用于对代码进行静态分析，检查代码中的错误、潜在问题以及风格违规。它类似于其他静态分析工具（如 `flake8` 或 `pylint`），主要功能如下：

**代码错误检查**

- 检查语法错误、未定义的变量、未使用的导入等问题。
- 例如，`ruff check` 可以检测到未使用的变量或导入模块。

**代码风格检查**

- 根据 PEP 8 等 Python 代码风格指南，检查代码格式问题。
- 例如，检查缩进、行长度、空格使用等。

**潜在问题检测**

- 检测可能导致运行时错误的代码模式。
- 例如，未处理的异常、不安全的操作等。

**实战一下**

```shell
import time, datetime

def hello():
    print("hello world!")
```

我们可以看到 导入的库，没有使用到。

```shell
uvx ruff check main.py
# 你会看到抛出很多错误
```

修复它

```shell
uvx ruff check --fix main.py
```

查看结果

```shell

def hello():
    print("hello world!")
```

你会看到未使用的库不见了。

## 规则

Ruff 支持 800 多个 lint 规则，其中许多规则受到 Flake8、isort、pyupgrade 等流行工具的启发。无论规则的来源如何，Ruff 都会将每条规则重新实现为 Rust 的第一方功能。

### 设置常用规则

```shell
[tool.ruff.lint]
select = [
    "E", # pycodestyle errors
    "W", # pycodestyle warnings
    "F", # pyflakes
    "I", # isort
    "B", # flake8-bugbear
    "C4", # flake8-comprehensions
    "UP", # pyupgrade
]
```

**让我们逐条解释一下，这些配置项的作用**

#### `"E"` - pycodestyle 错误

作用：启用 `pycodestyle` 工具中定义的错误检查规则。

示例规则：

- `E101`：缩进错误（例如混合使用空格和制表符）。
- `E112`：期望缩进的代码块没有缩进。
- `E113`：意外的缩进。

用途：帮助确保代码符合 PEP 8 的基本格式要求。

#### `"W"` - pycodestyle 警告

作用：启用 `pycodestyle` 工具中定义的警告检查规则。

示例规则：

- `W291`：行尾有多余的空格。
- `W292`：文件末尾缺少换行符。
- `W293`：空白行包含空格。

用途：检查代码中的格式问题，这些问题通常不会导致程序错误，但会影响代码的可读性。

#### `"F"` - pyflakes

作用：启用 `pyflakes` 工具中定义的规则，用于检查代码中的逻辑错误和潜在问题。

示例规则：

- `F401`：导入的模块未使用。
- `F821`：未定义的变量。
- `F841`：定义了但未使用的局部变量。

用途：帮助发现代码中的低级错误和潜在问题。

#### `"I"` - isort

作用：启用 `isort` 工具中定义的规则，用于检查导入语句的格式和排序。

示例规则：

- `I001`：导入语句未按字母顺序排序。
- `I002`：导入语句未分组（例如标准库、第三方库、本地库）。

用途：确保导入语句的格式一致且符合规范。

#### `"B"` - flake8-bugbear

作用：启用 `flake8-bugbear` 工具中定义的规则，用于检查代码中的潜在错误和不良实践。

示例规则：

- `B002`：在 `except` 语句中使用了裸异常（`except:`）。
- `B006`：在函数参数中使用了可变默认值（如 `def func(arg=[])`）。
- `B007`：定义了但未使用的循环变量。

用途：帮助避免常见的编程陷阱和不良实践。

#### `"C4"` - flake8-comprehensions

作用：启用 `flake8-comprehensions` 工具中定义的规则，用于检查列表、字典和集合推导式的使用。

示例规则：

- `C400`：不必要的列表推导式（可以直接使用生成器表达式）。
- `C401`：不必要的集合推导式。
- `C402`：不必要的字典推导式。

用途：优化推导式的使用，使代码更简洁高效。

#### `"UP"` - pyupgrade

作用：启用 `pyupgrade` 工具中定义的规则，用于检查并建议升级到更新的 Python 语法。

示例规则：

- `UP006`：建议使用 `class MyClass:` 而不是 `class MyClass(object):`（Python 3 中不需要显式继承 `object`）。
- `UP007`：建议使用 `TypeVar` 而不是 `typing.TypeVar`。
- `UP008`：建议使用 `super()` 而不是 `super(MyClass, self)`。

用途：帮助代码保持与最新 Python 版本的兼容性，并利用新特性。

### 设置忽略文件及文件夹

通过配置 `exclude`，你可以排除那些不需要被 Ruff 处理的文件或目录，从而减少不必要的检查或格式化操作。

```yaml
# 排除的文件或目录
exclude = [
    "build/",       # 构建输出目录
    "dist/",        # 分发文件目录
    "*.egg-info/",  # Python 包的元数据目录
    "__pycache__/", # Python 缓存文件目录
    ".pytest_cache/", # pytest 缓存目录
    ".mypy_cache/", # mypy 缓存目录
    ".venv/",       # 虚拟环境目录
    "venv/",        # 虚拟环境目录
    "env/",         # 虚拟环境目录
    "tests/data/",  # 测试数据目录
    "**/*.json",    # 所有 JSON 文件
    "**/*.csv",     # 所有 CSV 文件
    "**/*.xml",     # 所有 XML 文件
    "migrations/",  # Django 迁移文件目录
    "**/generated/", # 自动生成的代码目录
    "vendor/",      # 第三方库目录
    "lib/",         # 第三方库目录
    ".*",           # 所有隐藏文件和目录
    ".git/",        # Git 目录
    ".github/",     # GitHub 配置目录
    ".idea/",       # PyCharm 配置目录
    ".vscode/",     # VS Code 配置目录
    "**/__init__.py", # 所有 __init__.py 文件
    "setup.py",     # 项目的 setup.py 文件
    "requirements.txt", # 依赖文件
    "*.log",        # 所有日志文件
    "**/*.ipynb",   # 所有 Jupyter Notebook 文件
]
```

## 配合预提交

`pre-commit`是一个用于管理和运行 Git 预提交钩子（Git pre-commit hooks）的框架。它允许你在代码提交到 Git 仓库之前自动运行一系列检查或任务，例如代码格式化、静态分析、测试等，以确保代码质量和一致性。

### 准备工作

```shell
uv add pre-commit
uvx pre-commit installl
uvx pre-commit install
```

### 设置 pre-commit 配置文件

```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: check-added-large-files
      - id: check-toml
      - id: check-yaml
        args:
          - --unsafe
  - repo: https://github.com/charliermarsh/ruff-pre-commit
    rev: v0.2.2
    hooks:
      - id: ruff
        args:
          - --fix
      - id: ruff-format

ci:
  autofix_commit_msg: 🎨 [pre-commit.ci] Auto format from pre-commit.com hooks
  autoupdate_commit_msg: ⬆ [pre-commit.ci] pre-commit autoupdate
```

当我们使用 `git commit`的时候就会自动根据 `Ruff` 配置规则，检查 `python` 代码。

## 参考链接

- [Ruff 官方文档](https://docs.astral.sh/ruff/)
