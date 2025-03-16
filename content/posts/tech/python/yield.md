---
title: "Python yield 解析：原理、示例与 send 方法"
date: 2025-03-16T17:06:01+08:00
lastmod: 2025-03-16T17:06:01+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "python中的yield关键字详解"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202503161716254.png"
---

## 简介

`yield` 是 Python 中用于定义生成器函数的关键字

### 示例

```python
def simple_generator():
    yield 1
    yield 2
    yield 3

# 使用生成器
gen = simple_generator()

print(next(gen))  # 输出: 1
print(next(gen))  # 输出: 2
print(next(gen))  # 输出: 3
print(next(gen))  # StopIteration:
```

- 直接调用生成器函数无法获取结果，需要使用 `next()`方法
- 使用 `__next__()`方法等效

```python
gen.__next__()  # StopIteration:
```

### 作用

**暂停函数执行**：当函数执行到 `yield` 语句时，函数会暂停执行，并将 `yield` 后面的值返回给调用者

**保存函数状态**：函数的状态（包括局部变量、指令指针等）会被保存下来，下次调用时可以从暂停的地方继续执行

**生成值**：每次调用生成器的 `next()` 方法或使用 `for` 循环时，生成器会从上次暂停的地方继续执行，直到遇到下一个 `yield` 或函数结束

是不是不好理解 😂

## 详解

### 简单的示例 next

```python
def my_generator():
    print("Start")
    res1 = yield 1
    print(res1)
    res2 = yield 2
    print(res2)
    print("End")

gen = my_generator()  # 创建生成器对象，但不会执行
print(next(gen))
print(next(gen))
print(next(gen))

# 结果
# Start
# 1
# None
# 2
# None
# End
# StopIteration:
```

### 单步调试

1. 生成生成器对象 `gen`
2. `next` 方法调用 `gen`,生成器对象开始执行，输出 `Start`、返回 `1`遇到 `yield`暂停
3. 再使用 `next`、输出 `res1`，返回 `2`遇到 `yield` 暂停
4. 再调用 `next` 方法，输出 `res2`、输出 `End`、抛异常

### 同样的示例 send

我们写一个和上面示例一样的示例，但是不使用 `next`方法，而是使用 `send`方法

```python
def my_generator():
    print("Start")
    res1 = yield 1
    print(res1)
    res2 = yield 2
    print(res2)
    print("End")

gen = my_generator()  # 创建生成器对象，但不会执行
print(gen.send(None))
print(gen.send(None))
print(gen.send(None))

# 结果
# Start
# 1
# None
# 2
# None
# End
# StopIteration:
```

你会发现，结果和上面一样

### 分析

`**send**`** 的作用**

传递值：`send` 方法可以将一个值传递给生成器，这个值会成为当前暂停的 `yield` 表达式的结果

恢复执行：`send` 方法会从生成器暂停的地方继续执行，直到遇到下一个 `yield` 或生成器结束

返回值：`send` 方法会返回生成器通过 `yield` 生成的下一个值

### 不同的 send 值

```python
def my_generator():
    print("Start")
    res1 = yield 1
    print(res1)
    res2 = yield 2
    print(res2)
    print("End")

gen = my_generator()  # 创建生成器对象，但不会执行
print(gen.send(None))
print(gen.send("Fuck"))
print(gen.send("Fuck"))

# Start
# 1
# Fuck
# 2
# Fuck
# End
# StopIteration:
```

你看，接收的值不一样啦~

> 注意，你不能一开始就使用 send 方法发送非空值，这会导致抛出异常。错误原因：生成器还没有开始执行 `yield`，无法接收 `send("Func")` 传来的值

**结论**

`next(gen)` 等价于 `gen.send(None)`

至于说 `send` 方法中是不是调用了 `next` 方法，这个我没有调研，但 `deepseek` 和 `chatgpt` 都告诉我没有，只是内部逻辑相同 😅 有知道的同学可以分享一下~

## 连接

[python 中 yield 的用法详解——最简单，最清晰的解释](https://blog.csdn.net/mieleizhi0522/article/details/82142856)
