---
title: "Python 生成器与异步生成器"
date: 2025-04-16T11:50:07+08:00
lastmod: 2025-04-16T11:50:07+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Python 生成器与异步生成器 入门"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image/icons8-python-500.png" # 文章的图片
---

## 生成器

### 简介

生成器是一种惰性迭代器，它每次只生成一个值，用完了再生成下一个，而不是一次性生成全部所有值。一般有两种创建方式

### 创建生成器

- yield

```python
def count_up_to(max):
    count = 1
    while count <= max:
        yield count  # 返回当前值，暂停函数执行
        count += 1

gen = count_up_to(3)
for i in gen:
    print(i)
```

- 生成器表达式

```python
gen = (x * x for x in range(5))
for i in gen:
    print(i)
```

### yield 的作用

可以看我的这篇博客

[Python yield 解析：原理、示例与 send 方法](https://www.xxrbear.cn/posts/tech/python/yield/)

### 生成器的作用

一般用来解决大数据量大问题，例如，一口气生成一个百万数据的列表，都放到内存中会太消耗内存，这时候就可以使用生成器来逐步生成，降低内存占用率。又或者读取数据库时数据量太大也可以使用生成器，一样的道理。

### 生成器的高级用法

`send()`：将值传入生成器

```python
def custom_gen():
    x = yield "准备好了"
    yield f"你刚才说的是：{x}"

g = custom_gen()
print(next(g))        # 输出：准备好了
print(g.send("你好")) # 输出：你刚才说的是：你好
```

注意：不能一开始就使用`send`方法发送非 None 值，这样会报错。需要用 next 或者 send(None)启动生成器。

```plain
TypeError: can't send non-None value to a just-started generator
```

`throw()`：主动在 `yield` 处抛出异常

```python
def example():
    try:
        yield
    except ValueError:
        print("接住了异常")

g = example()
next(g)
g.throw(ValueError)
```

`close()`：停止生成器执行

```python
def mygen():
    try:
        while True:
            yield 1
    finally:
        print("生成器关闭")

g = mygen()
next(g)
g.close()
```

## 异步生成器

### 简介

异步生成器是 python3.6 之后增加的，它解决的问题和生成器基本一致

### 创建异步生成器

```python
async def async_gen():
    for i in range(3):
        await asyncio.sleep(1)  # 模拟异步操作
        yield i  # 注意：这是一个 async generator
```

- 调用异步生成器

```python
import asyncio

async def main():
    async for item in async_gen():
        print(item)

asyncio.run(main())
```

### 不支持高级方法

不支持 `send()`、`throw()`、`close()`
