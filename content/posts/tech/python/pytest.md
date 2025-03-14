---
title: "玩转 pytest：Python 测试的必备利器"
date: 2025-03-14T11:12:59+08:00
lastmod: 2025-03-14T11:12:59+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "pytest 是一个高效、灵活的 Python 测试框架，提供强大功能，帮助开发者提升测试效率和代码质量"
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

## 一、简介

pytest 是 Python 生态中最流行的测试框架之一，提供强大的功能、简洁的语法和丰富的插件系统

### 1.1 安装

```shell
uv add pytest
```

### 1.2 查看版本

```shell
uv run pytest --version

pytest 8.3.5
```

### 1.3 基本用法

```python
# test_sample.py
def add(x, y):
    return x + y

def test_add():
    assert add(2, 3) == 5

# 运行
uv run pytest
```

- 测试文件应以 `test_*.py` 或 `*_test.py` 命名
- 测试函数应以 `test_` 开头，否则 `pytest` 不会自动发现

### 1.4 测试报告

```python
uv run pytest -v    # 显示详细信息
uv run pytest -q    # 仅显示成功/失败
```

## 二、Fixtures

在`pytest` 中，`Fixtures` 常被直接称为 “测试夹具” 或 “测试装置”，用于为测试提供必要的资源、状态或环境，并在测试完成后进行清理

作用: Fixtures 用于为测试提供数据、状态或对象。它们可以在测试函数、类或模块中共享

生命周期: Fixtures 可以控制资源的创建和销毁，支持不同的作用域（`function`、`class`、`module`、`session`）

依赖注入: `pytest` 会自动将 fixtures 注入到测试函数中，只需在测试函数的参数中声明即可

```python
import pytest

@pytest.fixture
def sample_data():
    return {"name": "Alice", "age": 30}

# test_example.py
def test_sample_data(sample_data):
    assert sample_data["name"] == "Alice"
```

### 2.1 全局配置文件

`conftest.py` 是 `pytest` 的全局/局部配置文件，主要用于：

提供 `fixtures` 共享，避免在多个测试文件重复定义 `fixtures`

1. `conftest.py` 在 `pytest` 运行时自动加载，不需要手动 `import`
2. 作用范围取决于 `conftest.py` 所在的目录（层级越深，作用范围越小）

**示例：一个全局的测试数据库连接**

```python
# tests/conftest.py
import pytest

@pytest.fixture
def db_connection():
    print("\n🔌 Connecting to database...")
    yield "Database Connection"
    print("\n🔌 Closing database connection...")
```

## 三、参数化测试

在 `pytest` 中，参数化测试是一种非常强大的功能，可以让你用不同的输入数据运行同一个测试函数，从而避免重复代码。

参数化测试通过 `@pytest.mark.parametrize` 装饰器实现

```python
import pytest

@pytest.mark.parametrize("a, b, expected", [
    (1, 2, 3),
    (4, 5, 9),
    (10, 20, 30),
])
def test_addition(a, b, expected):
    assert a + b == expected

# 运行
uv run pytest
```

## 四、标记测试

### 4.1 跳过测试

```python
@pytest.mark.skip(reason="跳过此测试")
def test_skip():
    assert False
```

### 4.2 条件跳过

```python
import sys
import pytest

@pytest.mark.skipif(sys.platform == "win32", reason="不支持 Windows 平台")
def test_platform_specific():
    assert True

# 跳过 windows 平台上的测试
```

### 4.3 标记慢测试

```python
@pytest.mark.slow
def test_long_running():
    import time
    time.sleep(5)

# 可以选择性运行
uv run pytest -m "not slow"
```

## 五、处理异常

`pytest.raises` 是 `pytest` 提供的一个上下文管理器，用于断言代码是否抛出指定异常

```python
import pytest

def divide(a, b):
    return a / b

def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(1, 0)
```

## 六、Mock 场景

### 6.1 安装

```python
uv add pytest-mock
```

### 6.2 替换内置函数

```python
import time

def slow_function():
    time.sleep(5)  # 模拟耗时操作
    return "done"

def test_slow_function(mocker):
    mocker.patch("time.sleep", return_value=None)  # 替换 time.sleep，不让它真正执行
    assert slow_function() == "done"
```

### 6.3 替换类方法

```python
class DataFetcher:
    def get_data(self):
        return "Real Data"

def process():
    fetcher = DataFetcher()
    return fetcher.get_data()

def test_process(mocker):
    mocker.patch.object(DataFetcher, "get_data", return_value="Mocked Data")
    assert process() == "Mocked Data"
```

## 七、测试覆盖率

```shell
uv add pytest-cov
uv run pytest --cov=your_module
```
