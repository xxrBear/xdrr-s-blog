---
title: "Python asyncio 入门实战-1"
date: 2025-04-13T20:18:13+08:00
lastmod: 2025-04-13T20:18:13+08:00
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
summary: "Python asyncio 入门实战-1"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502261419829.jpg" # 文章的图片---
---

## 协程函数简介

### 前置知识

想了解关于 Python 并发编程的前置知识，例如什么是线程和进程、并发和并行的区别、asyncio 的事件循环是什么等，可以看本人的前篇博客

[一文粗通 Python asyncio 并发编程](https://www.xxrbear.cn/posts/tech/base/concurrent/)

### 定义一个协程函数

如果你不明白什么是 Python 的协程函数，你可以把它当作一个普通函数，但是它可以暂停和运行，而且普通函数的定义方式是 `def`协程函数的定义方式是 `async def`而暂停的方式是 `await`

**一个最简单的协程函数**

```python
async def hello() -> None:
    print("Hello")
```

### 运行一个协程函数

如何运行协程函数？你不能像普通函数那样，通过直接调用来运行协程函数，那样你只会得到一个协程对象

```python
In [3]: hello()
Out[3]: <coroutine object hello at 0x00000194E4671840>
```

你需要使用 asyncio 的 run 方法

```python
asyncio.run(hello())
```

至于 run 方法后面做了什么，我们后面会解释

### 使用 await 等待协程函数的结果

await 关键字后面通常会跟上协程函数（严格来说是可等待对象，后面会解释是什么），使得后面的协程函数开始运行，等待协程函数的结果。

```python
import asyncio


async def add(x, y) -> None:
    return x + y


async def main():
    result1 = await add(1, 2)  # 等待 add(1, 2) 的结果
    result2 = await add(10, 2)  # 等待 add(10, 2) 的结果
    print(result1)
    print(result2)


asyncio.run(main())
```

## 模拟耗时协程函数

### asyncio.sleep

`asyncio.sleep` 是一个非常重要的协程函数，我们通常使用它来模拟耗时的协程函数任务，例如查询数据库数据，调用 Web 接口等。

因为它是一个协程函数，这意味着使用它时，其他协程等待时，其他代码可以运行

举个例子，这个例子使用了 Task，如果不理解没关系后面会解释

```python
import time
import asyncio


async def add(x, y) -> None:
    await asyncio.sleep(3)
    return x + y


async def main():
    start = time.time()

    task1 = asyncio.create_task(add(1, 2))
    task2 = asyncio.create_task(add(1, 10))
    result1 = await task1
    result2 = await task2
    print(result1)
    print(result2)

    end = time.time()
    print(end - start)


asyncio.run(main())
```

运行这段代码，你会发现总耗时在三秒左右

### 为什么不能使用 time.sleep

你或许会疑惑，为什么我们不能使用 `time.sleep` 函数来模拟耗时协程函数。

使用 `time.sleep` 会阻塞当前事件循环，使得其他协程函数无法运行

```python
import asyncio
import time


async def task(name):
    print(f"{name} 开始")
    time.sleep(3)
    print(f"{name} 结束")


async def main():
    start = time.time()

    task1 = asyncio.create_task(task("你好"))
    task2 = asyncio.create_task(task("世界"))
    await task1
    await task2

    end = time.time()
    print(end - start)


asyncio.run(main())
```

执行这个函数大概会运行 6 秒，阻塞了！

## Task

任务是协程的包装器，它安排协程尽快在事件循环中运行，如上所示，任务中的等待时间都几乎是同时发生，即两个三秒耗时等待的协程，使用任务执行时间耗费在三秒出头。

### 创建一个任务

创建一个任务，我们需要使用方法 `asyncio.create_task`

```python
import asyncio
import time


async def task(name):
    print(f"{name} 开始")
    await asyncio.sleep(10)
    print(f"{name} 结束")


async def main():
    start = time.time()

    task1 = asyncio.create_task(task("你好"))
    task2 = asyncio.create_task(task("世界"))
    await task1
    await task2

    end = time.time()
    print(end - start)


asyncio.run(main())
```

### 取消任务

如果我们有这样一个需求，超过 5 秒的任务，我们不让它运行，直接取消。

```python
import asyncio


async def task(name):
    print(f"{name} 开始")
    await asyncio.sleep(10)
    print(f"{name} 结束")


async def main():

    task1 = asyncio.create_task(task("你好"))
    await asyncio.wait_for(task1, timeout=5)


asyncio.run(main())
```

## 可等待对象

通过前文我们知道，await 关键字后面可以接 task 和 coroutine，它们的共同点就是它们都是可等待对象。

但是下面我们还要介绍一个可等待对象 Future，你或许在编码中不会使用它，但理解它是我们了解 asyncio 内部工作原理的关键。

### 关于 future

future 是一个 Python 对象，它包含一个你希望在未来获取但现在可能还不存在的值。如果你使用过 JavaScript，你可能觉得它很像期约

### 创建一个 future

```python
from asyncio import Future

future = Future()

print(f"Future 状态完成了嘛，{future.done()}")

future.set_result(1)

print(f"现在呢 {future.done()}")
print(f"Future 的result是什么 {future.result()}")
```

```python
E:\src\demo\main.py:3: DeprecationWarning: There is no current event loop
  future = Future()

Future 状态完成了嘛，False
现在呢 True
Future 的result是什么 1
```

### 可等待对象之间的关系

事实上

- Task 继承自 Future
- Future 和 Coroutine 都继承自 awaitable 抽象基类

## 异步函数的陷阱

### 简介

并不是所有任务使用协程都会提升效率，例如 CPU 密集型任务或者阻塞式 IO 的库

### CPU 密集型

```python
import time
import asyncio
import functools


def async_timed():
    """计算异步函数耗时"""

    def wrapper(func):
        @functools.wraps(func)
        async def wrapped(*args, **kwargs):
            print(f"开始函数 {func} 参数是 {args} {kwargs}")
            start = time.time()
            try:
                return await func(*args, **kwargs)
            finally:
                end = time.time()
                total = end - start
                print(f"完成函数 {func} 共计 {total:.4f} 秒")

        return wrapped

    return wrapper


@async_timed()
async def cpu_bound_work():
    count = 0
    for i in range(10000000):
        count += i
    return count


@async_timed()
async def main():

    task_one = asyncio.create_task(cpu_bound_work())
    task_two = asyncio.create_task(cpu_bound_work())
    await task_one
    await task_two


asyncio.run(main())
```

返回结果

```python
开始函数 <function main at 0x0000027A6543B7E0> 参数是 () {}
开始函数 <function cpu_bound_work at 0x0000027A634CC900> 参数是 () {}
完成函数 <function cpu_bound_work at 0x0000027A634CC900> 共计 0.4046 秒
开始函数 <function cpu_bound_work at 0x0000027A634CC900> 参数是 () {}
完成函数 <function cpu_bound_work at 0x0000027A634CC900> 共计 0.4049 秒
完成函数 <function main at 0x0000027A6543B7E0> 共计 0.8096 秒
```

你看每个任务花费大概 0.4 秒左右，但总耗时确实两倍，这说明 CPU 密集型任务无法使用 asyncio 来减少耗时。

原因是 asyncio 使用的是单线程并发模型，它还是没法突破 GIL 的限制。

看上面的例子，可能还是无法解释 CPU 密集型任务对异步函数的限制，让我们再举一个例子

```python
import time
import asyncio
import functools


def async_timed():
    """计算异步函数耗时"""

    def wrapper(func):
        @functools.wraps(func)
        async def wrapped(*args, **kwargs):
            print(f"开始函数 {func} 参数是 {args} {kwargs}")
            start = time.time()
            try:
                return await func(*args, **kwargs)
            finally:
                end = time.time()
                total = end - start
                print(f"完成函数 {func} 共计 {total:.4f} 秒")

        return wrapped

    return wrapper


@async_timed()
async def cpu_bound_work():
    count = 0
    for i in range(10000000):
        count += i
    return count


@async_timed()
async def main():

    task_one = asyncio.create_task(cpu_bound_work())
    task_two = asyncio.create_task(cpu_bound_work())
    task_three = asyncio.create_task(asyncio.sleep(3))
    task_four = asyncio.create_task(asyncio.sleep(3))
    await task_one
    await task_two
    await task_three
    await task_four


asyncio.run(main())
```

输出

```python
开始函数 <function main at 0x0000017CC874B7E0> 参数是 () {}
开始函数 <function cpu_bound_work at 0x0000017CC67CC900> 参数是 () {}
完成函数 <function cpu_bound_work at 0x0000017CC67CC900> 共计 0.4142 秒
开始函数 <function cpu_bound_work at 0x0000017CC67CC900> 参数是 () {}
完成函数 <function cpu_bound_work at 0x0000017CC67CC900> 共计 0.4049 秒
完成函数 <function main at 0x0000017CC874B7E0> 共计 3.8243 秒
```

运行上面的例子，你会发现耗时 3.8 秒左右，并不是 3 秒左右，也就是说，当前事件循环会被阻塞，IO 密集型的也会受到影响。

当然，上面的例子，调换一下顺序可以优化

```python
import time
import asyncio
import functools


def async_timed():
    """计算异步函数耗时"""

    def wrapper(func):
        @functools.wraps(func)
        async def wrapped(*args, **kwargs):
            print(f"开始函数 {func} 参数是 {args} {kwargs}")
            start = time.time()
            try:
                return await func(*args, **kwargs)
            finally:
                end = time.time()
                total = end - start
                print(f"完成函数 {func} 共计 {total:.4f} 秒")

        return wrapped

    return wrapper


@async_timed()
async def cpu_bound_work():
    count = 0
    for i in range(10000000):
        count += i
    return count


@async_timed()
async def main():


    task_three = asyncio.create_task(asyncio.sleep(3))
    task_four = asyncio.create_task(asyncio.sleep(3))

    task_one = asyncio.create_task(cpu_bound_work())
    task_two = asyncio.create_task(cpu_bound_work())
    await task_three
    await task_four
    await task_one
    await task_two


asyncio.run(main())
```

输出

```python
开始函数 <function main at 0x000002A2D7BBB7E0> 参数是 () {}
开始函数 <function cpu_bound_work at 0x000002A2D5C8C900> 参数是 () {}
完成函数 <function cpu_bound_work at 0x000002A2D5C8C900> 共计 0.3849 秒
开始函数 <function cpu_bound_work at 0x000002A2D5C8C900> 参数是 () {}
完成函数 <function cpu_bound_work at 0x000002A2D5C8C900> 共计 0.3900 秒
完成函数 <function main at 0x000002A2D7BBB7E0> 共计 3.0197 秒
```

由此看来，事件循环的执行顺序是先添加的任务先执行呐

### 阻塞型的库

requests 是一个阻塞型的 http 请求库

```python
import time
import asyncio
import functools

import requests


def async_timed():
    """计算异步函数耗时"""

    def wrapper(func):
        @functools.wraps(func)
        async def wrapped(*args, **kwargs):
            print(f"开始函数 {func} 参数是 {args} {kwargs}")
            start = time.time()
            try:
                return await func(*args, **kwargs)
            finally:
                end = time.time()
                total = end - start
                print(f"完成函数 {func} 共计 {total:.4f} 秒")

        return wrapped

    return wrapper


@async_timed()
async def cpu_bound_work():
    count = 0
    for i in range(10000000):
        count += i
    return count


@async_timed()
async def get_example_status():
    return requests.get("https://www.example.com").status_code


@async_timed()
async def main():
    task_1 = asyncio.create_task(get_example_status())
    task_2 = asyncio.create_task(get_example_status())

    await task_1
    await task_2


asyncio.run(main())
```

输出

```python
开始函数 <function main at 0x000001B8ABAABC40> 参数是 () {}
开始函数 <function get_example_status at 0x000001B8ABAABB00> 参数是 () {}
完成函数 <function get_example_status at 0x000001B8ABAABB00> 共计 0.2488 秒
开始函数 <function get_example_status at 0x000001B8ABAABB00> 参数是 () {}
完成函数 <function get_example_status at 0x000001B8ABAABB00> 共计 0.1691 秒
完成函数 <function main at 0x000001B8ABAABC40> 共计 0.4181 秒
```

## 事件循环

### 简介

我们都知道，事件循环是 asyncio 的核心，我们可能都会用 `asyncio.run` 来运行应用程序，并在幕后创建事件循环，但有的时候我们可能想自己创建事件循环，使用低层的方法。

### 手动创建事件循环

```python
import asyncio

async def main():
    await asyncio.sleep(1)


loop = asyncio.new_event_loop()

try:
    loop.run_until_complete(main())
finally:
    loop.close()
```
