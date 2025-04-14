---
title: "用 Python 从零构建异步回显服务器"
date: 2025-04-14T22:16:16+08:00
lastmod: 2025-04-14T22:16:16+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "一起动手用 python 构建一个 TCP 服务器"
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

让我们从 0 开始，搭建一个异步服务输出服务器。

### 套接字

套接字（socket），是不同计算机中实现通信的一种方式，你可以理解成一个接口，它会在客户端和服务端建立连接，一台发送数据，一台接收数据，靠的就是套接字。

### 阻塞套接字服务器

```python
import socket

# socket.AF_INET: IPv4 主机号 + 端口号
# socket.SOCK_STREAM: TCP 连接
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# socket.SOL_SOCKET: 套接字选项
# socket.SO_REUSEADDR: 允许重用本地地址,避免端口被占用
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ('localhost', 8000)
server_socket.bind(server_address) # 绑定地址和端口号
# socket.listen(): 监听连接请求
server_socket.listen()

connection, client_address = server_socket.accept()
print(f'我获取了一个连接地址: {client_address}')
```

启动后，尝试连接服务器

```python
# 启动服务器

python server.py

# 连接服务器
nc localhost 8000

# 输出
我获取了一个连接地址: ('127.0.0.1', 60525)
```

### 从套接字读取和写入数据

前面的示例只能接收一次连接，而且无法读取和写入数据，让我们修改一下

```python
import socket

# socket.AF_INET: IPv4 主机号 + 端口号
# socket.SOCK_STREAM: TCP 连接
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# socket.SOL_SOCKET: 套接字选项
# socket.SO_REUSEADDR: 允许重用本地地址,避免端口被占用
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ('localhost', 8000)
server_socket.bind(server_address) # 绑定地址和端口号
# socket.listen(): 监听连接请求
server_socket.listen()

try:
    connection, client_address = server_socket.accept()
    print(f'我获取了一个连接地址: {client_address}')

    buffer = b''

    while buffer[-5:] != b'quit\n':
        data = connection.recv(1024)
        if not data:
            break
        else:
            buffer += data
            print(f'接收数据: {data}')

    print(f'接收到的所有数据: {buffer}')
    connection.sendall(buffer)

finally:
    server_socket.close()


```

这里最重要的就是 recv 方法，它可以从套接字中接收数据，并且写入服务端，然后使用 sendall 方法将接收到的数据，最后全部发送给客户端。

### 允许多个连接的服务器

如果你尝试使用多个客户端连接上面的服务器，你会发现只有第一个会生效，让我们改写一下，让它可以支持连接多个客户端。

```python
import socket

# socket.AF_INET: IPv4 主机号 + 端口号
# socket.SOCK_STREAM: TCP 连接
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# socket.SOL_SOCKET: 套接字选项
# socket.SO_REUSEADDR: 允许重用本地地址,避免端口被占用
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ('localhost', 8000)
server_socket.bind(server_address) # 绑定地址和端口号
# socket.listen(): 监听连接请求
server_socket.listen()

connections = []

try:
    while True:
        connection, client_address = server_socket.accept()
        print(f'我获取了一个连接地址: {client_address}')

        buffer = b''

        while buffer[-5:] != b'quit\n':
            data = connection.recv(1024)
            if not data:
                break
            else:
                buffer += data
                print(f'接收数据: {data}')

        print(f'接收到的所有数据: {buffer}')
        connection.sendall(buffer)

finally:
    server_socket.close()
```

对比一下代码，你会发现，只不过是把每一个连接放入一个列表中，然后循环遍历接收数据，但是这段示例依然不够优秀，当运行时你会发现，每次第二次连接都会阻塞，都会要第一次连接断开后，第二次连接才会生效。

原因是因为我们使用的是阻塞套接字，想要允许多个客户端连接，我们需要使用非阻塞套接字。

## 使用非阻塞套接字

python 套接字中设置非阻塞套接字比较简单，一行代码就搞定了`setblocking(False)`

```python
import socket

# socket.AF_INET: IPv4 主机号 + 端口号
# socket.SOCK_STREAM: TCP 连接
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# socket.SOL_SOCKET: 套接字选项
# socket.SO_REUSEADDR: 允许重用本地地址,避免端口被占用
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ('localhost', 8000)
server_socket.bind(server_address) # 绑定地址和端口号
# socket.listen(): 监听连接请求
server_socket.listen()
server_socket.setblocking(False) # 设置为非阻塞模式

connections = []

try:
    while True:
        try:
            connection, client_address = server_socket.accept()
            connection.setblocking(False)
            print(f'我获取了一个连接地址: {client_address}')
            connections.append(connection)
            print(f'当前连接数: {len(connections)}')
        except BlockingIOError:
            pass

        for connection in connections:
            try:
                buffer = b''

                while buffer[-5:] != b'quit\n':
                    data = connection.recv(1024)
                    if not data:
                        break
                    else:
                        buffer += data
                        print(f'接收数据: {data}')

                print(f'接收到的所有数据: {buffer}')
                connection.send(buffer)
            except BlockingIOError:
                pass

finally:
    server_socket.close()
```

尝试运行，会发现 cpu 飙到 100% ，我们需要更好的方法。

## 使用 selectors 模块优化

Python 的选择器模块是连接操作系统的低层接口，cpu 消耗很少。

```python
import selectors
import socket

selector = selectors.DefaultSelector()

server_socket = socket.socket()
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

server_address = ('127.0.0.1', 8000)
server_socket.setblocking(False)
server_socket.bind(server_address)
server_socket.listen()

selector.register(server_socket, selectors.EVENT_READ)

while True:
    events = selector.select(timeout=1)

    if len(events) == 0:
        print('没有事件，等待一会吧')

    for event, _ in events:
        event_socket = event.fileobj

        if event_socket == server_socket:
            client_socket, client_address = server_socket.accept()
            print(f'连接来自 {client_address}')
            client_socket.setblocking(False)
            selector.register(client_socket, selectors.EVENT_READ)
        else:

            data = event_socket.recv(1024)

            if data:
                print(f'我获取了数据 {data}')
                event_socket.send(b'You Got ' + data)
            else:
                print('客户端断开连接')
                selector.unregister(event_socket)
                event_socket.close()
```

代码解释

```python
# 创建选择器（自动选择适合当前操作系统的机制）
selector = selectors.DefaultSelector()

# 创建服务器 socket 并设置参数
server_socket = socket.socket()
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

# 注册服务器 socket，监听“可读事件” (EVENT_READ)
# 意思是：当有客户端连接进来，我们就会收到事件通知。
selector.register(server_socket, selectors.EVENT_READ)

# 返回所有“发生了事件的 socket 对象”，等待最多 1 秒；
events = selector.select(timeout=1)
```

因为 selectors 模块是操作系统级别的高效事件通知系统，所以使用这个程序它的 CPU 使用率很低。

Python 自带的 `asyncio` 默认事件循环基于 `selectors` 模块

## 使用 asyncio 构建

### 回显服务器

```python
import asyncio
import socket

async def echo(connection, loop):
    while data := await loop.sock_recv(connection, 1024):
        await loop.sock_sendall(connection, data)

async def listen_for_connection(server_socket, loop):
    while True:
        connection, address = await loop.sock_accept(server_socket)
        connection.setblocking(False)
        print(f'获取到连接: {address}')
        asyncio.create_task(echo(connection, loop))

async def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    server_address = ('localhost', 8000)
    server_socket.setblocking(False)
    server_socket.bind(server_address)
    server_socket.listen()

    await listen_for_connection(server_socket, asyncio.get_event_loop())

asyncio.run(main())
```

`:=`是 python3.8 新增的海象运算符

```python
while data := await ...

相当于：

data = await ...
while data:
```

### 解决服务器的错误任务

```python
import asyncio
import socket
import logging


async def echo(connection, loop):
    try:
        while data := await loop.sock_recv(connection, 1024):
            if data == b'boom\n':
                raise Exception('Boom!')
            await loop.sock_sendall(connection, data)
    except Exception as e:
        logging.exception(f'服务器报错啦: {e}')
    finally:
        connection.close()
        print('关闭连接')

async def listen_for_connection(server_socket, loop):
    while True:
        connection, address = await loop.sock_accept(server_socket)
        connection.setblocking(False)
        print(f'获取到连接: {address}')

        asyncio.create_task(echo(connection, loop))

async def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    server_address = ('localhost', 8000)
    server_socket.setblocking(False)
    server_socket.bind(server_address)
    server_socket.listen()

    await listen_for_connection(server_socket, asyncio.get_event_loop())

asyncio.run(main())
```
