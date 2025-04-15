---
title: "Python asyncio 入门实战-2"
date: 2025-04-15T20:23:36+08:00
lastmod: 2025-04-15T20:23:36+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Python asyncio 入门实战-2"
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

## 简介

第一部分看这里

[Python asyncio 入门实战-1](https://www.xxrbear.cn/posts/tech/python/python-asyncio-inaction-1/)

第一部分我们学习了 asyncio 的协程函数、任务、Future 等知识，第二部分我们介绍一下 aiohttp 异步非阻塞 http 请求库的基本使用和 Python 中处理异步任务时最常用的几种函数：`asyncio.gather()`、`asyncio.wait()`、`asyncio.as_completed()`

## 前置代码

### 计算协程函数花费时间的代码

```python
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
```

## aiohttp 的使用方法

第一部分我们说过，python 最常用的 http 请求库之一`requests`库是阻塞的，调用它会阻塞事件循环中的其他任务，而`aiohttp`就是非阻塞的库，它可以在事件循环中以非阻塞的形式发送 http 请求。

### 发送 Web 请求

使用`pip install aiohttp`安装

```python
import asyncio
import aiohttp
from aiohttp import ClientSession

import functools
import time

@async_timed()
async def fetch_status(session: ClientSession, url: str) -> int:
    """Fetch the status code of a URL."""
    async with session.get(url) as response:
        return response.status

@async_timed()
async def main() -> None:
    async with aiohttp.ClientSession() as session:
        url = 'https://example.com'
        status_code = await fetch_status(session, url)
        print(f"Status code for {url}: {status_code}")

asyncio.run(main())
```

### 设置超时时间

aiohttp 的请求超时时间为 5 分钟，我们可以手动设置的低一点。

```python
import asyncio
import aiohttp
from aiohttp import ClientSession

import functools
import time

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
async def fetch_status(session: ClientSession, url: str) -> int:
    """Fetch the status code of a URL."""
    async with session.get(url) as response:
        return response.status

@async_timed()
async def main() -> None:
    session_timeout = aiohttp.ClientTimeout(total=0.1, connect=1)
    async with aiohttp.ClientSession(timeout=session_timeout) as session:
        url = 'https://example.com'
        status_code = await fetch_status(session, url)
        print(f"Status code for {url}: {status_code}")

asyncio.run(main())
```

## 执行并发请求

### 任务并发 web 请求

在第一部分，我们学到了可以使用 Task 来执行协程函数，但是这有一个问题，就是如果需要同时执行数个任务的话，会比较麻烦，你可能会想到使用循环，但是这样有个陷阱

```python
@async_timed()
async def main() -> None:
    delay_times = [3, 3, 3]
    [await asyncio.create_task(asyncio.sleep(delay)) for delay in delay_times]

asyncio.run(main())
```

执行代码你会发现这会耗时超过 9 秒，也就意味着它是串行执行的，至于为什么，是因为它没创建一个任务就`await`这意味着阻塞，稍微修改一下就好了

```python
@async_timed()
async def main() -> None:
    delay_times = [3, 3, 3]
    tasks = [ asyncio.create_task(asyncio.sleep(delay)) for delay in delay_times]
    [await task for task in tasks]

asyncio.run(main())
```

这样就是先统一创建任务，然后运行了。

虽然这能解决一次性创建多个任务的需求，但还是有缺点的，最大的缺点就是会有失败的请求，如果一个请求失败了，前面已经完成的结果我们无法拿到并且使用。

### 使用 gather

为了解决上面提到的问题，我们需要学习一个新的方法，`asyncio.gather`它直接翻译为 收集，顾名思义，收集一系列可等待对象执行

```python
@async_timed()
async def main() -> None:
    urls = ['https://www.example.com'] * 1000
    request_list = [fetch_status(url) for url in urls]
    status_code = await asyncio.gather(*request_list)
    print(status_code)

asyncio.run(main())
```

对比一下，比同步省下不少时间，接下来，我们要着重介绍 gather 的不同用法了。

### 结果的有序性

```python
async def task(name, delay):
    await asyncio.sleep(delay)
    print(f"Task {name} done (after {delay}s)")
    return f"Result from {name}"

async def main():
    results = await asyncio.gather(
        task("A", 3),
        task("B", 1),
        task("C", 2),
    )
    print("\nFinal results (ordered):")
    print(results)
```

如果你运行上面这段示例，会返回如下的结果

```python
Task B done (after 1s)
Task C done (after 2s)
Task A done (after 3s)

Final results (ordered):
['Result from A', 'Result from B', 'Result from C']
```

我们看到，任务 A 耗时最长，但是还是在第一位，也就是说，gather 返回的结果默认是按顺序的。

### 抛出异常的两种方式

```python
@async_timed()
async def main():
    async with aiohttp.ClientSession() as session:
        urls = ['https://www.example.com', 'bad']
        tasks = [fetch_status(session, url) for url in urls]
        status_code = await asyncio.gather(*tasks)
        print(f"Status codes: {status_code}")
```

默认方式是直接返回异常调用栈，同时也不会取消其他正在运行的任务。

我们可以通过设置`return_exceptions`参数来修改默认方式。

```python
@async_timed()
async def main():
    async with aiohttp.ClientSession() as session:
        urls = ['https://www.example.com', 'bad']
        tasks = [fetch_status(session, url) for url in urls]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        exceptions = [res for res in results if isinstance(res, Exception)]
        success = [res for res in results if not isinstance(res, Exception)]
        print(f"All results: {results}")
        print(f"Exceptions: {exceptions}")
        print(f"Success: {success}")
```

这样，我们就可以针对成功的结果和异常的结果分别处理，虽然这样还是有点笨笨的，没关系，后面我们还会介绍一种方法。

### 在请求完成时立即处理

gather 方法有一个问题，就是它的结果必须要全部执行完成之后才可用，我们需要一个可以在每个任务完成时，都可以使用这些已完成任务结果的方法，巧了，`as_completed`就是这样一个方法。

```python
@async_timed()
async def main():
    async with aiohttp.ClientSession() as session:
        fetchers = [
            fetch_status(session, 'https://www.example.com', delay=1),
            fetch_status(session, 'https://www.example.org', delay=2),
            fetch_status(session, 'https://www.example.net', delay=3)
        ]
        for f in asyncio.as_completed(fetchers):
            status = await f
            print(f"Status: {status}")
```

设置超时时间

```python
@async_timed()
async def main():
    async with aiohttp.ClientSession() as session:
        fetchers = [
            fetch_status(session, 'https://www.example.com', delay=1),
            fetch_status(session, 'https://www.example.org', delay=2),
            fetch_status(session, 'https://www.example.net', delay=3)
        ]
        for f in asyncio.as_completed(fetchers):
            status = await f
            print(f"Status: {status}")
```

## 使用 wait 控制

通过前面的内容，我们知道了`gather`和`as_completed`的一些缺点，那就是没有简单的方法来取消正在运行的任务。而`wait`提供了更具体的控制来处理成功或者失败的请求，它返回两个集合，第一个集合是完成或者失败的任务结果集合，第二个是还没有执行的任务的集合。

### 等待所有任务完成

默认情况下，wait 会返回所有任务已完成的结果

```python
import asyncio

async def task(name, delay):
    await asyncio.sleep(delay)
    print(f"Task {name} done")
    return f"Result from {name}"

@async_timed()
async def main():
    tasks = [
        asyncio.create_task(task("A", 2)),
        asyncio.create_task(task("B", 1)),
        asyncio.create_task(task("C", 3)),
    ]

    # wait for all to complete
    done, pending = await asyncio.wait(tasks)

    print("\nDone tasks:")
    for d in done:
        print(d.result())  # 获取每个已完成任务的返回值

asyncio.run(main())
```

### 异常处理

```python
@async_timed()
async def main():
    async with aiohttp.ClientSession() as session:
        good_result = fetch_status(session, "https://www.example.com")
        bad_result = fetch_status(session, "bad.com")

        fetchers = [asyncio.create_task(good_result), asyncio.create_task(bad_result)]

        done, pending = await asyncio.wait(fetchers)

        for done_task in done:
            if done_task.exception() is None:
                print(done_task.result())
            else:
                logging.error("请求失败", exc_info=done_task.exception())
```

我们可以使用 `exception`方法处理异常。我们可以再加一段代码，使得当发生异常时，后面的任务都取消

```python
@async_timed()
async def main():
    async with aiohttp.ClientSession() as session:
        good_result = fetch_status(session, "https://www.example.com")
        bad_result = fetch_status(session, "bad.com")

        fetchers = [asyncio.create_task(good_result), asyncio.create_task(bad_result)]

        done, pending = await asyncio.wait(fetchers)

        for done_task in done:
            if done_task.exception() is None:
                print(done_task.result())
            else:
                logging.error("请求失败", exc_info=done_task.exception())
```

### 及时处理完成的任务

参数 return_when 默认是 `asyncio.ALL_COMPLETED`即完成所有任务，遇到异常时返回参数是`asyncio.FIRST_EXCEPTION`，现在我们介绍至少有一个结果时返回，它的参数是 `asyncio.FIRST_COMPLETED`

```python
async def main():
    async with aiohttp.ClientSession() as session:

        fetchers = [
            asyncio.create_task(fetch_status(session, "https://www.example.com")),
            asyncio.create_task(fetch_status(session, "https://www.example.com")),
            asyncio.create_task(fetch_status(session, "https://www.example.com")),
        ]

        done, pending = await asyncio.wait(
            fetchers, return_when=asyncio.FIRST_COMPLETED
        )

        print(f"当前完成任务数量 {len(done)}")
        print(f"当前未完成任务数量 {len(pending)}")

        for done_task in done:
            print(await done_task)
```

它会返回一个已完成任务，和两个待执行任务。
