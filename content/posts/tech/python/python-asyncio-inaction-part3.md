---
title: "Python asyncio 入门实战-3"
date: 2025-04-17T10:53:55+08:00
lastmod: 2025-04-17T10:53:55+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Python asyncio 入门实战-3"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502261419829.jpg" # 文章的图片
---

## 简介

第一二部分在这里

[Python asyncio 入门实战-1](https://www.xxrbear.cn/posts/tech/python/python-asyncio-inaction-1/)

[Python asyncio 入门实战-2](https://www.xxrbear.cn/posts/tech/python/python-asyncio-inaction-2/)

之前我们已经学会了 Python asyncio 等基本概念和基本使用方法。

这一部分我们将着重学习 `multiprocessing`库的基本使用，进程池的作用，以及如何与 `asyncio`配合处理 CPU 密集型任务，还有如何在多进程任务间加锁以避免竟态条件等。

## 多进程

### multiprocessing 入门

使用多进程处理一个 CPU 密集型任务

```python
import time
from multiprocessing import Process


def count(count_to: int) -> int:
    start = time.time()
    counter = 0
    while counter < count_to:
        counter = counter + 1
    end = time.time()
    print(f'Finished counting to {count_to} in {end-start}')
    return counter


if __name__ == "__main__":
    start_time = time.time()

    to_one_hundred_million = Process(target=count, args=(100000000,))
    to_two_hundred_million = Process(target=count, args=(200000000,))

    to_one_hundred_million.start()
    to_two_hundred_million.start()

    to_one_hundred_million.join() # 等待主进程
    to_two_hundred_million.join() # 等待主进程

    end_time = time.time()
    print(f'Completed in {end_time-start_time}')
```

### 使用进程池

上面的示例，我们得不到函数的返回结果，还需要使用 start 和 join 来辅助多进程执行。

使用进程池方法，不需要使用这些

```python
from multiprocessing import Pool


def say_hello(name: str) -> str:
    return f'Hi there, {name}'


if __name__ == "__main__":
    with Pool() as process_pool:  # A
        hi_jeff = process_pool.apply(say_hello, args=('Jeff',))  # B
        hi_john = process_pool.apply(say_hello, args=('John',))
        print(hi_jeff)
        print(hi_john)
```

但是这个方法也有一个很大的缺点就是，每个任务执行的时候是阻塞的，即如果每个任务如果都是耗时 10s，那么两个就是 20s ，并没有效率的提升，接下来我们使用 `apply_async`来解决这个问题。

```python
from multiprocessing import Pool


def say_hello(name: str) -> str:
    return f'Hi there, {name}'


if __name__ == "__main__":
    with Pool() as process_pool:
        hi_jeff = process_pool.apply_async(say_hello, args=('Jeff',))
        hi_john = process_pool.apply_async(say_hello, args=('John',))
        print(hi_jeff.get())
        print(hi_john.get())
```

虽然上面的示例可以提升多任务效率，但是还是有个小问题，那就是如果 `hi_jeff`执行花费 10s 而 `hi_john`执行花费 1 s，那么 `get`方法会阻塞 10s 然后几乎同一时间返回两个结果。

没事，我们有办法解决这个问题，那就是进程池执行器。

## 进程池执行器

Python 的 concurrent.futures 里封装了进程池执行器和线程池执行器，让我们先学习进程池执行器。

### 简单应用

```python
import time
from concurrent.futures import ProcessPoolExecutor


def count(count_to: int) -> int:
    start = time.time()
    counter = 0
    while counter < count_to:
        counter = counter + 1
    end = time.time()
    print(f'Finished counting to {count_to} in {end - start}')
    return counter


if __name__ == "__main__":
    with ProcessPoolExecutor() as process_pool:
        numbers = [1, 3, 5, 22, 100000000]
        for result in process_pool.map(count, numbers):
            print(result)
```

实例化 `ProcessPoolExecutor`对象会默认创建与系统相同核心数量的进程，map 方法类似 `asyncio.as_completed`方法

执行结果

```python
Finished counting to 1 in 1.1920928955078125e-06
Finished counting to 3 in 9.5367431640625e-07
Finished counting to 5 in 7.152557373046875e-07
Finished counting to 22 in 1.1920928955078125e-06
1
3
5
22
Finished counting to 100000000 in 3.007368803024292
100000000
```

缺点是，它还是按顺序返回结果的，也就是说，如果高用时任务在前面，后面的任务返回结果还是阻塞的，在高用时任务返回后才会一起返回。

### 带异步事件循环的执行器

```python
import asyncio
from asyncio.events import AbstractEventLoop
from concurrent.futures import ProcessPoolExecutor
from functools import partial
from typing import List


def countdown(count_from: int) -> int:
    counter = 0
    while counter < count_from:
        counter = counter + 1
    return counter


async def main():
    with ProcessPoolExecutor() as process_pool:
        loop: AbstractEventLoop = asyncio.get_running_loop()
        nums = [1, 3, 5, 22, 100000000]
        calls: List[partial[int]] = [partial(countdown, num) for num in nums]
        call_coros = []

        for call in calls:
            call_coros.append(loop.run_in_executor(process_pool, call))

        results = await asyncio.gather(*call_coros)

        for result in results:
            print(result)


if __name__ == "__main__":
    asyncio.run(main())
```

`run_in_executor`接收一个可调用对象，并且不能带参数。然后返回一个 awaitable 对象，可以被 asyncio 接管使用。例如上面的 gather 方法。

## 多进程中的锁

### 共享数据

在多进程编程中，每个进程通常有自己的内存空间，普通变量不能在进程之间共享。如果你想让多个进程共享某个变量（比如一个计数器、布尔值、状态标志等），就可以用 `Value` 来实现。

### 竞态条件

当一组操作的结果取决于哪个操作先完成时，就会出现竞态条件，例如

```python
from multiprocessing import Process, Value

def increment_value(shared_value):
    # Increment the shared value by 1
    shared_value.value = shared_value.value + 1

if __name__ == "__main__":
    for _ in range(100):
        integer = Value('i', 0)
        procs = [
            Process(target=increment_value, args=(integer,)),
            Process(target=increment_value, args=(integer,))
        ]

        [p.start() for p in procs]
        [p.join() for p in procs]
        print(integer.value)
        print(integer.value == 2)
```

预期的结果是 2，但是最终的结果并不总是 2，这种就是竞态条件。那么为什么不总是 2 呢？原因是，进程 1 和进程 2，读取 value 值的时间是不一定的，可能 value 已经完成了+1 那么读取就是 1，如果两个进程读取 value 都是 0，那么最终结果就是 1。为了解决这个问题，我们需要加锁

### 使用锁进行同步

为了避免进程间的竞态条件，我们需要在进程间加锁，也就是说，当一个进程先读取到 value 的值的时候，加锁，这样其他的进程就会等待锁释放才能读取到 value 的值，避免了静态条件。

```python
from multiprocessing import Process, Value

def increment_value(shared_value):
    # Increment the shared value by 1
    with shared_value.get_lock():  # Ensure thread-safe access
        shared_value.value += 1

if __name__ == "__main__":
    for _ in range(100):
        integer = Value('i', 0)
        procs = [
            Process(target=increment_value, args=(integer,)),
            Process(target=increment_value, args=(integer,))
        ]

        [p.start() for p in procs]
        [p.join() for p in procs]
        print(integer.value)
        print(integer.value == 2)
```

这可以修复净态条件，但可能降低应用程序性能。

### 进程池共享数据

```python
from concurrent.futures import ProcessPoolExecutor
import asyncio
from multiprocessing import Value

shared_counter: Value

def init(counter: Value):
    """Initialize the shared counter."""
    global shared_counter
    shared_counter = counter

def increment_value():
    # Increment the shared value by 1
    with shared_counter.get_lock():  # Ensure thread-safe access
        shared_counter.value += 1

async def main():
    counter = Value('i', 0)  # Shared counter initialized to 0
    with ProcessPoolExecutor(initializer=init, initargs=(counter,)) as pool:
        await asyncio.get_running_loop().run_in_executor(pool, increment_value)
        print(counter.value)
```

我们不能直接在 init 函数中初始化 shared_value 的值为 Value('i', 0) 那样会创建多个线程池，同时初始化多个 shared_value 值为 0 导致错误结果。

## 多进程，多时间循环

虽然 multiprocessing 模块主要用来解决 CPU 密集型任务，但是也可以为 IO 密集型任务带来性能提升，因为一些 IO 密集型任务中也存在大量的 CPU 运算。

感兴趣的可自行 google 或者 chatGPT 和 Deepseek😅

或者看《Python asyncio 并发编程》这本书的第 6 章节。
