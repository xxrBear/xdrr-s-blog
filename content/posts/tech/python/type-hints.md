---
title: "一文粗通 Python Type Hints 及静态代码检查工具"
date: 2025-03-18T12:38:45+08:00
lastmod: 2025-03-18T12:38:45+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Python 类型提示与 MyPy：让动态语言拥有静态语言的超能力！"
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

### 什么是 Type Hints

在 `Python` 中，`Type Hints` 一般可以翻译为以下几种方式，它们基本上是同一个概念：

1. **类型注解**（最常见，强调“注解”是代码的一部分）
2. **类型标注**（也较常见，强调“标注”是对变量的补充信息）
3. **类型提示**（较少用，但依然有人使用，强调它是“提示”而非强制）
4. **类型标识**（较少见，但也有人使用，类似于标注的意思）

无论你使用哪种翻译，意思都是指 **Python 代码中对变量、参数、返回值等添加的类型信息，以便进行静态检查和提高可读性**

鄙人习惯称其为 类型提示

### 静态类型检查工具

`Python` 是动态类型语言，变量的类型是在运行时确定的。因此需要静态类型检查工具的的帮助让我们检查代码，发现运行前潜在的类型错误，减少运行时错误

常见的 `Python` 静态类型检查工具

- mypy
- pyright
- pylint

本篇博文主要介绍 mypy

**示例**

- 安装 mypy

```shell
uv add mypy
```

- 添加示例

```python
def add(x: int, y: int) -> int:  # 这里的 int 就是类型提示
    return x +


add("x", 1)
```

- 运行 mypy

```python
mypy main.py

main.py:5: error: Argument 1 to "add" has incompatible type "str"; expected "int"  [arg-type]
Found 1 error in 1 file (checked 1 source file)
```

可以看到这里函数输入一个字符串，但是期望的是一个整型，这里 `mypy` 就会提示了。

## 类型提示的意义

### 提高代码可读性

在没有类型提示的情况下，阅读代码时，我们无法直接知道变量或函数参数的类型，可能会写出错误的代码。

```python
def add(a, b):
    return a + b
```

使用类型提示

```python
def add(a: int, b: int) -> int:
    return a + b
```

- 明确数据类型，开发者一眼就能看出 `a` 和 `b` 预期是 `int`
- 减少歧义，让代码更易理解

### 提前发现错误

`Python` 是动态类型语言，如果没有类型检查，错误往往会在运行时才暴露

```python
def greet(name):
    return "Hello, " + name

print(greet(42))  # 运行时报错：TypeError: can only concatenate str (not "int") to str
```

使用类型检查工具（如 `mypy`）可以提前发现错误

- 静态检测：错误可以在运行时之前发现
- 减少 bug，避免不必要的调试时间

### IDE 支持

当代码有类型注解时，IDE（如 PyCharm、VS Code）可以提供

- 代码补全
- 类型推断
- 错误提示
- 参数提示

### 便于团队协作

在大型项目中，不同开发者可能会对变量的类型理解不同：

```python
def process_data(data):
    ...
```

- `data` 是 `list[str]` 还是 `dict`？
- 如果团队成员不了解 `data` 的类型，可能会误用。

```python
from typing import List

def process_data(data: List[str]) -> None:
    ...
```

- 让团队成员更快理解代码
- 减少错误，避免误传参数

### 让 Python 拥有静态语言的优势

许多静态语言（如 Java、C++）都有强类型检查，可以

- 防止低级错误
- 提升运行效率
- 更容易重构

`Python` 没有强制类型检查，但 `type hints`+ `mypy` 让 `Python` 部分具备静态语言的优势，且仍然保持灵活性

## MyPy 入门

### 安装 mypy

```python
uv add mypy
```

uv 是一个超快的 Python 包和解释器管理工具，如果你不了解，可以看下面的这篇博文，或者直接使用 `pip`安装也可以

[uv](https://www.baidu.com)

### 示例

```python
def add(a: int, b: int) -> int:
    return a + b

result = add(3, "hello")  # ❌ 传入了 str 而不是 int
```

使用命令行运行 mypy

```python
uv run mypy main.py
```

报错

```plain
ain.py:4: error: Argument 2 to "add" has incompatible type "str"; expected "int"  [arg-type]
Found 1 error in 1 file (checked 1 source file)
```

期待一个整型，但是输入的是 str

## 常用的类型注解

### 内置类型

- `int`
- `float`
- `None`
- `str`
- `bool`
- `bytes`

### 容器类型

`Python` 提供了 **List、Tuple、Set、Dict** 等数据结构，在 `MyPy` 中需要显式标注内部元素的类型。

```python
from typing import List, Tuple, Dict, Set

numbers: List[int] = [1, 2, 3]
coordinates: Tuple[float, float] = (10.5, 20.3)
student_ages: Dict[str, int] = {"Alice": 25, "Bob": 22}
unique_ids: Set[int] = {1, 2, 3, 4}
```

**注意**：`Python 3.9` 以后，可以直接使用 `list[int]`、`dict[str, int]` 等简化写法，而不需要 `typing.List`

### Optional

有时候，一个函数的返回值可能是字符串也可能是 `None` 这时候我们就可以使用 `Optional` 注解

**举例**

```python
from typing import Optional

def find_user(user_id: int) -> Optional[str]:
    if user_id == 1:
        return "Alice"
    return None
```

`Optional` 表示参数可以是 `None`

相同写法(`python3.10` 及以上)

```python
def find_user(user_id: int) -> str | None:
```

### Union

`Union` 类型表示变量可以是多个可能的类型

```python
from typing import Union

value: Union[int, str] = 42
value = "hello"  # ✅ 允许
```

相同写法(`python3.10` 及以上)

```python
value: int | str = 42
```

### Any

`Any` 表示可以是任意类型，`MyPy` 不会检查它。

```python
from typing import Any

def dynamic_function(data: Any) -> Any:
    return data
```

### Final

`Final` 是用于类型注解的一个特殊标记，表示某个变量、属性或方法的值是不可变的（即不可重新赋值或覆盖）。它通常用于标记常量或不可更改的值。

```python
from typing import Final


class Constants:
    PI: Final = 3.14159  # 不能被修改


Constants.PI = 2.2


# main.py:8: error: Cannot assign to final attribute "PI"  [misc]
# Found 1 error in 1 file (checked 1 source file)
```

### Callable

`Callable` 用于标注函数或可调用对象的类型。它可以明确函数的参数类型和返回值类型，使代码更具可读性和安全性。

**用法**

```python
Callable[[参数类型1, 参数类型2, ...], 返回值类型]
```

**举例**

```python
from typing import Callable

# 定义一个函数，接受两个 int 参数，返回一个 int
def add(x: int, y: int) -> int:
    return x + y

# 使用 Callable 标注函数类型
math_operation: Callable[[int, int], int] = add

# 调用函数
result = math_operation(10, 20)
print(result)  # 输出: 30

# 使用 Callable 标注 lambda 函数
square: Callable[[int], int] = lambda x: x ** 2

# 调用函数
print(square(5))  # 输出: 25
```

### 迭代器和生成器类型

**<font style="color:rgb(64, 64, 64);">Iterator</font>**

迭代器是一个实现了 `__iter__` 和 `__next__` 方法的对象。`typing` 模块提供了 `Iterator[T]` 类型，用于标注迭代器的元素类型。

```python
from typing import Iterator

# 自定义迭代器类
class CountUpTo:
    def __init__(self, max_value: int):
        self.max_value = max_value
        self.current = 0

    def __iter__(self) -> Iterator[int]:
        return self

    def __next__(self) -> int:
        if self.current >= self.max_value:
            raise StopIteration
        self.current += 1
        return self.current

# 使用自定义迭代器
counter: Iterator[int] = CountUpTo(5)
for num in counter:
    print(num)

# 输出:
# 1
# 2
# 3
# 4
# 5
```

**<font style="color:rgb(64, 64, 64);">Generator</font>**

生成器是一种特殊的迭代器，使用 `yield` 关键字生成值。`typing` 模块提供了 `Generator[Y, S, R]` 类型，用于标注生成器的产出类型（`Y`）、发送类型（`S`）和返回类型（`R`）

```python
from typing import Generator


# 定义一个生成器，产出 int，发送 None，返回 str
def count_down_from(n: int) -> Generator[int, None, str]:
    while n > 0:
        yield n
        n -= 1
    return "Done"


# 使用生成器
counter: Generator[int, None, str] = count_down_from(3)

# 获取生成器的返回值
while True:
    try:
        num = next(counter)
        print(num)
    except StopIteration as e:
        print(e.value)  # 输出: Done
        break

# 输出:
# 3
# 2
# 1
# Done
```

### <font style="color:rgb(64, 64, 64);">Sequence</font>

`Sequence` 表示**有序的、可迭代的容器**，但不要求可变。

```python
from typing import Sequence

def print_elements(elements: Sequence[int]) -> None:
    for element in elements:
        print(element)

# 使用 list
print_elements([1, 2, 3])
# 输出:
# 1
# 2
# 3

# 使用 tuple
print_elements((4, 5, 6))
# 输出:
# 4
# 5
# 6

# 使用 str（如果元素类型是 str）
def print_chars(chars: Sequence[str]) -> None:
    for char in chars:
        print(char)

print_chars("abc")
# 输出:
# a
# b
# c
```

### Literal

`Literal` 用于限制变量或参数只能取 **特定的值**，提高类型安全性。

```python
from typing import Literal

def set_status(status: Literal["pending", "approved", "rejected"]) -> None:
    print(f"订单状态设置为: {status}")

set_status("pending")   # ✅ 合法
set_status("approved")  # ✅ 合法
set_status("rejected")  # ✅ 合法

# set_status("other")  # ❌ 报错：只能传入 "pending"、"approved" 或 "rejected"
```

### <font style="color:rgb(64, 64, 64);">泛型</font>

Python 的泛型（Generics）是一种 **类型参数化** 机制，它允许我们编写通用代码，使函数、类或数据结构适用于 **多种类型**，提高 **代码复用性** 和 **类型安全性**

**简单示例**

```python
from typing import TypeVar

T = TypeVar("T")  # 定义泛型类型 T

def first_element(items: list[T]) -> T:
    return items[0]

print(first_element([1, 2, 3]))  # int -> 1
print(first_element(["a", "b", "c"]))  # str -> "a"
```

函数`first_element`可以接收两种类型的列表

**定义泛型类**

```python
from typing import Generic

T = TypeVar("T")  # 泛型类型 T

class Box(Generic[T]):
    def __init__(self, content: T):
        self.content = content

    def get_content(self) -> T:
        return self.content

int_box = Box(100)  # T = int
str_box = Box("hello")  # T = str

print(int_box.get_content())  # 100
print(str_box.get_content())  # "hello"
```

## 配置 MyPy

### 简介

`MyPy` 允许你通过配置文件来定制类型检查的行为，从而更好地适应你的项目需求。配置文件通常是 `.ini` 格式，名为 `mypy.ini` 或 `setup.cfg`，并且可以用来控制 MyPy 的检查规则、严格性等

你也可以使用 uv 的配置文件 `pyproject.toml`配合 mypy 使用，这样就不用写 mypy.ini 文件了

### 配置示例

```toml
[project]
name = "app"
version = "0.1.0"
description = ""
requires-python = ">=3.10,<4.0"
dependencies = [
]

[tool.uv]
dev-dependencies = [
    "mypy<2.0.0,>=1.8.0"
]


[tool.mypy]
# 启用严格模式
strict = true

# 指定要检查的文件或目录
files = "main.py"

# 忽略缺少类型注解的导入
ignore_missing_imports = true

# 显示错误代码，便于查找问题
show_error_codes = true

# 排除某些文件或目录的检查
exclude = "^tests/.*"  # 排除 tests 目录中的文件

# 指定使用的 Python 版本
python_version = 3.13
```

### 测试

```shell
uv run mypy
```
