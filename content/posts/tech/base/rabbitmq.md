---
title: "RabbitMQ 入门"
date: 2025-02-28T13:12:38+08:00
lastmod: 2025-02-28T13:12:38+08:00
author: ["熊大如如"]
tags: # 标签
  - "rabbitmq"
description: ""
weight:
slug: ""
summary: "rabbitmq消息队列入门，python操作版本"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502281313775.jpg" # 文章的图片
---

## 一、消息队列

### 1.1 AMQP 是什么

AMQP：即（Advanced Message Queuing Protocol 高级消息队列协议），提供统一消息服务。是应用层协议的一个开放标准，为面向消息的中间件设计。`RabbitMQ` 是 AMQP 的 Erlang 语言实现。

### 1.2 消息队列是什么

消息队列从字面意思上来说，就是放置消息的队列，遵循先进先出原则。

### 1.3 为什么使用消息队列

**流量削峰**

当你的系统处理量不足时，你就可以将消息放在消息队列中做缓冲，然后逐步处理即可。

**异步处理**

当你需要调用一个用时很长的任务时，你可以使用异步调用的方式，然后让这个任务给消息队列发一个消息，这样你就不用一直等待了。

**消息广播**

比如消息订阅，我们只需要管消息有没有进入队列，下游有没有消费消息我们不用管。

## 二、安装

docker 一键安装

```python
# latest RabbitMQ 4.0.x
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management
```

[127.0.0.1:15672](http://127.0.0.1:15672)

默认账户密码是 `guest`，但是注意，这个账户密码只能在安装了`RabbitMQ`的机器上登录，如果你是服务器安装的，那么默认是不可以登录的，建议新建一个管理员用户，然后删除 `guest`用户。

## 三、核心概念

### 3.1 Virtual Host

虚拟主机是消息队列中非常重要的概念，它提供了资源隔离、权限控制和配置独立性，使得同一台服务器上的多个应用能够安全、独立地运行在自己的“虚拟环境”中。

### 3.2 Queue

`RabbitMQ` 的 `queue` 就是存储消息的地方

### 3.3 Exchange

交换机是 `RabbitMQ` 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息 **路由到队列**

### 3.4 Broker

在技术领域，`Broker` 通常翻译为 “代理” 或 “中间人”。在消息队列系统（如 `RabbitMQ`）中，`Broker` 指的是消息代理服务器，负责接收、存储和转发消息。  
 接收和分发消息的应用，`RabbitMQ Server` 就是 `Message Broker`

### 3.5 Connection

生产者-消费者 和 `Broker` 之间的 TCP 连接。

### 3.6 Channel

如果每一次访问 `RabbitMQ` 都建立一个 `Connection`，在消息量大的时候建立 TCP 连接的开销将是巨大的，效率也较低。`Channel` 作为轻量级的 `Connection` 极大减少了操作系统建立 `TCP connection` 的开销。

### 3.7 Binding

`exchange`和`queue`之间的虚拟连接，`binding`中可以包含`routing key`，`Binding`信息被保存到`exchange`中的查询表中，用于`message`的分发依据。

### 3.8 生产者-消费者

生产消息和消费消息的应用程序。

### 3.9 一图胜千言

简化图

![](https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202502281313051.png)

生产者生产消息，推送到交换机中，交换机路由到队列，消费者消费。

## 四、交换机和交换机类型

### 4.1 简介

服务器发送消息不会直接发送到队列中，只能将消息发送给交换机，然后根据确定的规则，`RabbitMQ`将会决定消息该投递到哪个队列。这些规则称为路由键（routing key），队列通过路由键绑定到交换机上。消息发送到服务器端，消息也有自己的路由键（也可以是空），`RabbitMQ` 也会将消息和消息指定发送的交换机的绑定（binding，就是队列和交互机的根据路由键映射的关系）的路由键进行匹配。

### 4.2 Direct Exchange（直连交换机）

作用：将消息路由到 `Binding Key` 和 `Routing Key` 完全匹配的队列。

使用场景：适用于消息一对一精确传递的场景。

示例：

- 生产者发送消息时指定 `Routing Key = "order"`。
- 队列绑定到交换机时指定 `Binding Key = "order"`。
- 只有 `Routing Key` 和 `Binding Key` 完全匹配时，消息才会被路由到该队列。

### 4.3 Fanout Exchange（扇形交换机）

作用：将消息广播到所有绑定到该交换机的队列，忽略 `Routing Key`。

使用场景：适用于消息广播的场景，例如日志系统或通知系统。

示例：

- 生产者发送消息到 `Fanout Exchange`
- 所有绑定到该交换机的队列都会收到消息，无论 `Routing Key` 是什么。

### 4.4 Topic Exchange（主题交换机）

作用：根据 `Routing Key` 和 `Binding Key` 的模式匹配来路由消息。`Binding Key` 支持通配符：

- `*` 匹配一个单词。
- `#` 匹配零个或多个单词。

使用场景：适用于需要灵活路由规则的场景，例如根据消息类型或来源进行路由。

示例：

- 生产者发送消息时指定 `Routing Key = "order.create"`。
- 队列绑定到交换机时指定 `Binding Key = "order.*"`。
- 消息会被路由到该队列，因为 `Routing Key` 匹配 `Binding Key` 的模式。

### 4.5 Headers Exchange（头交换机）

作用：根据消息的 `Headers` 属性（键值对）进行路由，而不是 `Routing Key`。绑定队列时可以指定匹配规则（`x-match`）：

- `x-match = all`：`Headers` 必须完全匹配所有键值对。
- `x-match = any`：`Headers` 只需匹配任意一个键值对。

使用场景：适用于需要基于消息属性进行复杂路由的场景。

示例：

- 生产者发送消息时指定 `Headers = {"type": "order", "status": "new"}`。
- 队列绑定到交换机时指定 `x-match = all` 和 `Headers = {"type": "order", "status": "new"}`。
- 消息会被路由到该队列，因为`Headers`完全匹配。

### 4.6 Default Exchange（默认交换机）

作用：`RabbitMQ`默认创建的交换机，是一个特殊的`Direct Exchange`。所有队列都会自动绑定到该交换机，`Binding Key` 为队列名称。

使用场景：适用于简单的消息传递场景。

示例：

- 生产者发送消息时指定 `Routing Key = "queue_name"`。
- 消息会被路由到名为 `queue_name` 的队列。

## 五、入门实战

### 5.1 Hello World

**生产者**

```python
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.queue_declare(queue='admin的队列')

channel.basic_publish(exchange='', routing_key='hello', body='Hello World!')
print(" [x] Sent 'Hello World!'")
connection.close()
```

**消费者**

```python
#!/usr/bin/env python
import pika, sys, os

def main():
    connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
    channel = connection.channel()

    channel.queue_declare(queue='hello')

    def callback(ch, method, properties, body):
        print(f" [x] Received {body}")

    channel.basic_consume(queue='hello', on_message_callback=callback, auto_ack=True)

    print(' [*] Waiting for messages. To exit press CTRL+C')
    channel.start_consuming()

if __name__ == '__main__':
    try:
        main()
    except KeyboardInterrupt:
        print('Interrupted')
        try:
            sys.exit(0)
        except SystemExit:
            os._exit(0)
```

### 5.2 工作队列

在这个教程中，我们将创建一个工作队列，用于在多个工作者之间分配耗时任务。

设置`prefetch_count=1` 这使用 `basic.qos` 协议方法告诉 `RabbitMQ` 一次不要给一个工作者发送多于一条消息。换句话说，不要在工作者处理并确认了上一条消息之前发送新的消息。相反，它会将其发送给下一个尚未忙碌的工作者。

**生产者**

```python
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.queue_declare(queue='task_queue', durable=True)

message = ' '.join(sys.argv[1:]) or "Hello World!"
channel.basic_publish(
    exchange='',
    routing_key='task_queue',
    body=message,
    properties=pika.BasicProperties(
        delivery_mode=pika.DeliveryMode.Persistent
    ))
print(f" [x] Sent {message}")
connection.close()
```

**消费者**

```python
#!/usr/bin/env python
import pika
import time

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.queue_declare(queue='task_queue', durable=True)
print(' [*] Waiting for messages. To exit press CTRL+C')


def callback(ch, method, properties, body):
    print(f" [x] Received {body.decode()}")
    time.sleep(body.count(b'.'))
    print(" [x] Done")
    ch.basic_ack(delivery_tag=method.delivery_tag)


channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='task_queue', on_message_callback=callback)

channel.start_consuming()
```

### 5.3 发布者-订阅者

上面的工作队列模式，让我们只能把消息发给一个消费者，现在我们要发送给每个消费者

**生产者**

```python
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='logs', exchange_type='fanout')

message = ' '.join(sys.argv[1:]) or "info: 我是顶顶顶!"
channel.basic_publish(exchange='logs', routing_key='', body=message)
print(f" [x] Sent {message}")
connection.close()
```

**消费者**

```python
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='logs', exchange_type='fanout')

result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

channel.queue_bind(exchange='logs', queue=queue_name)

print(' [*] Waiting for logs. To exit press CTRL+C')


def callback(ch, method, properties, body):
    print(f" [x] {body}")


channel.basic_consume(
    queue=queue_name, on_message_callback=callback, auto_ack=True)

channel.start_consuming()

```

### 5.4 路由

在 `RabbitMQ` 中，`Routing` 是指消息从 Exchange 传递到 `Queue` 的过程。路由的核心是通过 `Routing Key` 和 `Binding Key` 的匹配规则，决定消息应该被发送到哪些队列。

`Direct Exchange` 的路由算法非常简单：消息会被发送到与消息的`Routing Key`完全匹配的 `Binding Key` 的队列。

匹配规则：`Routing Key` 必须与 `Binding Key` 完全一致。

**生产者**

```python
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='direct_logs', exchange_type='direct')

severity = sys.argv[1] if len(sys.argv) > 1 else 'info'
message = ' '.join(sys.argv[2:]) or 'Hello World!'
channel.basic_publish(
    exchange='direct_logs', routing_key=severity, body=message)
print(f" [x] Sent {severity}:{message}")
connection.close()
```

**消费者**

```python
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='direct_logs', exchange_type='direct')

result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

severities = sys.argv[1:]
if not severities:
    sys.stderr.write("Usage: %s [info] [warning] [error]\n" % sys.argv[0])
    sys.exit(1)

for severity in severities:
    channel.queue_bind(
        exchange='direct_logs', queue=queue_name, routing_key=severity)

print(' [*] Waiting for logs. To exit press CTRL+C')


def callback(ch, method, properties, body):
    print(f" [x] {method.routing_key}:{body}")


channel.basic_consume(
    queue=queue_name, on_message_callback=callback, auto_ack=True)

channel.start_consuming()
```

### 5.5 主题

在`RabbitMQ`中，Topic Exchange 是一种灵活的路由模式，允许根据 `Routing Key`和`Binding Key`的模式匹配将消息路由到队列。主题模式的核心是支持通配符匹配，适用于需要根据消息的某些属性（如消息类型、来源等）进行动态路由的场景。

有需求可自行谷歌 🙂

### 5.6 RPC

远程过程调用

有需求可自行谷歌 🙂

## 六、参考链接

- [04-RabbitMQ 高级教程](https://janycode.github.io/2021/08/17/08_%E6%A1%86%E6%9E%B6%E6%8A%80%E6%9C%AF/07_RabbitMQ/04-RabbitMQ%20%E9%AB%98%E7%BA%A7%E6%95%99%E7%A8%8B/#1-MQ-%E7%9A%84%E7%9B%B8%E5%85%B3%E6%A6%82%E5%BF%B5)
- [RabbitMQ 官方文档](https://www.rabbitmq.com/tutorials)
