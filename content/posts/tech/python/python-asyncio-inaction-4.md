---
title: "Python asyncio 入门实战-4"
date: 2025-04-18T11:26:51+08:00
lastmod: 2025-04-18T11:26:51+08:00
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
summary: "Python Asyncio 第四部分"
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

[Python asyncio 入门实战-1](https://www.xxrbear.cn/posts/tech/python/python-asyncio-inaction-1/)

[Python asyncio 入门实战-2](https://www.xxrbear.cn/posts/tech/python/python-asyncio-inaction-2/)

[Python asyncio 入门实战-3](https://www.xxrbear.cn/posts/tech/python/python-asyncio-inaction-part3/)

这一部分我们学习多线程的相关知识，包含了多线程如何处理阻塞的库 requests、 如何与 asyncio 集合使用、线程锁等内容。

### 为什么使用多线程

你可能会想，IO 密集型任务我们使用`asyncio`不就可以了，为什么还要使用多线程呢？

其实在真实开发环境中，从 0 开始到新项目是很少见的，一般都是在别人的代码上进行修改，这个时候使用多线程去处理之间遗留的同步阻塞代码就很有用。

## Threading 模块

### 多线程回显服务器

```python
from threading import Thread
import socket

def echo(client: socket):
    while True:
        data = client.recv(2048)
        print(f"Received: {data}")
        client.sendall(data)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(('localhost', 8000))
    server.listen()
    while True:
        connection, _  = server.accept()
        thread = Thread(target=echo, args=(connection,))
        thread.start()
```

如果你运行这个示例你会发现，确实可以允许多个客户端的连接，但是也有问题，首先，如果你想退出客户端时，后台会一直接收空字节，造成 cpu 空转。还有如果主程序退出了，后台线程还在，会导致 Python 进程无法正常退出。

我们需要修改一下代码，修复这些问题

```python
from threading import Thread
import socket

class ClientEchoThread(Thread):

    def __init__(self, client):
        super().__init__()
        self.client = client

    def run(self):
        try:
            while True:
                data = self.client.recv(2048)
                if not data:
                    raise BrokenPipeError('连接关闭')
                print(f"Received: {data}")
                self.client.sendall(data)
        except OSError as e:
            print(f"多线程退出异常: {e}")

    def close(self):
        if self.is_alive():
            self.client.sendall(bytes('退出', encoding='utf-8'))
            self.client.shutdown(socket.SHUT_RDWR)


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind((('localhost', 8000)))
    server.listen()
    connection_threads = []

    try:
        while True:
            connection, addr  = server.accept()
            thread = ClientEchoThread(connection)
            connection_threads.append(thread)
            thread.start()
    except KeyboardInterrupt:
        print("Server shutting down...")
        [thread.close() for thread in connection_threads]
```

## asyncio 使用线程

### 线程池执行器

前面我们已经介绍了进程池执行器，现在我们介绍线程池执行器。其实线程池执行器的用法与进程池执行器的用法基本一致。

我们知道，python 的 http 请求库`requests`是阻塞的，即使用它请求多个网站，它是串行执行的，我们使用线程池执行器来改造一下，让`requests`可以支持并行任务。

```python
import time
import requests
from concurrent.futures import ThreadPoolExecutor

def get_status_code(url):
    try:
        response = requests.get(url)
        return response.status_code
    except requests.RequestException as e:
        return str(e)

start = time.time()

with ThreadPoolExecutor() as executor:
    urls = ['https://www.example.com' for _ in range(1000)]
    results = executor.map(get_status_code, urls)
    for result in results:
        print(result)

end = time.time()

print(f"Execution time: {end - start} seconds")
```

执行上面的代码，与阻塞的代码相比性能会大幅度提升，但是与 aiohttp 请求相比，你会发现还是不如它的性能快，这是因为，多线程的最大数量被设置为 32 个，也就是说，只能最大发送 32 个请求，让我们修改一下这个限制

```python
import time
import requests
from concurrent.futures import ThreadPoolExecutor

def get_status_code(url):
    try:
        response = requests.get(url)
        return response.status_code
    except requests.RequestException as e:
        return str(e)

start = time.time()

with ThreadPoolExecutor(max_workers=1000) as executor:
    urls = ['https://www.example.com' for _ in range(1000)]
    results = executor.map(get_status_code, urls)
    for result in results:
        print(result)

end = time.time()

print(f"Execution time: {end - start} seconds")
```

我们把最大线程改成了一千，运行一下，确实有不错的性能提升。

但是注意，并不是线程数量越多就越好，在开发环境，你还是要测试一下。从小线程开始，逐步提升测试，找到一个最佳的线程数量。

### 使用 asyncio 的线程池执行器

前面的示例，我们没有配合 asyncio 使用，这部分我们使用 asyncio 配合线程池执行器来完成特定任务。

```python
import time
import asyncio
import functools
from concurrent.futures import ThreadPoolExecutor

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

def get_status_code(url):
    try:
        response = requests.get(url)
        return response.status_code
    except requests.RequestException as e:
        return str(e)

# start = time.time()

@async_timed()
async def main():
    loop = asyncio.get_running_loop()
    with ThreadPoolExecutor(max_workers=1000) as executor:
        urls = ['https://www.example.com' for _ in range(1000)]
        tasks = [loop.run_in_executor(executor, functools.partial(get_status_code, url)) for url in urls]
        results = await asyncio.gather(*tasks)
        print(results)

asyncio.run(main())
```

厉不厉害，上面的例子中，我们在`run_in_executor`方法中手动的传入了线程池执行器，但其实它可以接收一个执行器参数 None，从而使用它的默认执行器，它的默认执行器就是线程池，所以上面的代码我们可以简写成这样：

```python
@async_timed()
async def main():
    loop = asyncio.get_running_loop()
    urls = ['https://www.example.com' for _ in range(1000)]
    tasks = [loop.run_in_executor(None, functools.partial(get_status_code, url)) for url in urls]
    results = await asyncio.gather(*tasks)
    print(results)
```

但这意味着我们不能传入最大线程了呀？

在 python3.9 之后，使用`to_thread`方法可以更简单的完成上面的代码

```python
@async_timed()
async def main():
    urls = ['https://www.example.com' for _ in range(1000)]
    tasks = [asyncio.to_thread(get_status_code, url) for url in urls]
    results = await asyncio.gather(*tasks)
    print(results)
```

## 多线程中的锁

### 锁是起什么作用的

在多线程程序里，如果多个线程同时修改同一份数据，就可能出现数据竞争的问题。

```python
import threading

count = 0

def add():
    global count
    for _ in range(1000000):
        count += 1

threads = []
for _ in range(2):
    t = threading.Thread(target=add)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(count)
```

加锁可以解决

```python
def add():
    global count
    for _ in range(1000000):
        with lock:
            count += 1
```

### 可重入锁

普通的 `Lock` 有个特点，同一个线程如果已经 `acquire()` 了锁，再次调用 `acquire()` 就会死锁自己，因为锁已经被自己占着了。

而可重入锁（RLock），允许同一个线程，多次获得同一把锁，不会死锁，只要 `release()` 相同次数就可以释放。

例如

```python
import threading

lock = threading.Lock()

def task():
    lock.acquire()
    print("第一次上锁")
    lock.acquire()   # 这里会死锁！！
    print("第二次上锁")
    lock.release()
    lock.release()

thread = threading.Thread(target=task)
thread.start()
thread.join()
```

运行到第二个 `acquire()` 的时候，程序卡死了

使用可重入锁就不会

```python
import threading

lock = threading.RLock()

def task():
    lock.acquire()
    print("第一次上锁")
    lock.acquire()
    print("第二次上锁")
    lock.release()
    lock.release()

thread = threading.Thread(target=task)
thread.start()
thread.join()
```

因为：

- RLock 内部维护了一个计数器。
- 第一次 `acquire()`，计数器 `+1`
- 第二次 `acquire()`，计数器再 `+1`
- 必须调用两次 `release()`，计数器归零，锁才真正释放

以后只要代码里出现一个锁、多层调用的结构，优先考虑 `RLock`，不会出错！

### 死锁

两个（或更多）线程互相占有对方需要的资源，并且互相等待，导致谁也无法继续执行，程序就卡死了

```python
import threading
import time

lockA = threading.Lock()
lockB = threading.Lock()

def task1():
    lockA.acquire()
    print("任务1 获得了锁A")
    time.sleep(1)
    lockB.acquire()
    print("任务1 获得了锁B")
    lockB.release()
    lockA.release()

def task2():
    lockB.acquire()
    print("任务2 获得了锁B")
    time.sleep(1)
    lockA.acquire()
    print("任务2 获得了锁A")
    lockA.release()
    lockB.release()

t1 = threading.Thread(target=task1)
t2 = threading.Thread(target=task2)

t1.start()
t2.start()

t1.join()
t2.join()
```

为了避免这种情况，我们可以统一加锁顺序

```python
def task1():
    with lockA:
        print("任务1 获得了锁A")
        time.sleep(1)
        with lockB:
            print("任务1 获得了锁B")

def task2():
    with lockA:  # 注意，先拿锁A再拿锁B
        print("任务2 获得了锁A")
        time.sleep(1)
        with lockB:
            print("任务2 获得了锁B")
```
