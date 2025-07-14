---
title: "Python Logging 详解"
date: 2025-07-14T16:00:12+08:00
lastmod: 2025-07-14T16:00:12+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "python日志模块"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cdn.jsdelivr.net/gh/xxrBear/image/icons8-python-500.png"  # 文章的图片
---
## 基础用法
```python
import logging

# 设置日志等级和输出格式
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s'
)

logging.debug("调试信息")
logging.info("普通信息")
logging.warning("警告")
logging.error("错误")
logging.critical("严重错误")
```

## 核心组成（五大组件）
```latex
Logger（日志记录器）     ← 你在代码中调用的接口
  ↓
Handler（日志处理器）    ← 决定日志输出到哪里（终端、文件等）
  ↓
Formatter（格式器）      ← 决定日志内容怎么显示
  ↓
Filter（过滤器）         ← 控制哪些日志可以通过
```

再配上一个统一配置模块（Config），构成完整的日志系统

## 日志等级
| 级别     | 方法                | 说明                             |
| -------- | ------------------- | -------------------------------- |
| DEBUG    | `logger.debug()`    | 调试信息，最详细                 |
| INFO     | `logger.info()`     | 一般信息，如启动日志、请求成功等 |
| WARNING  | `logger.warning()`  | 警告，但不影响程序执行           |
| ERROR    | `logger.error()`    | 错误，程序功能部分失效           |
| CRITICAL | `logger.critical()` | 严重错误，系统可能无法继续运行   |


## Handler 处理者
`Handler` 是 Python `logging` 模块的核心组成部分之一，用来决定日志记录的输出位置和输出方式

一个 Logger 可以绑定多个 Handler。每个 Handler 可单独设置输出等级、格式器、过滤器

### 内置 Handler
| 类名                       | 输出位置                | 特点                                      |
| -------------------------- | ----------------------- | ----------------------------------------- |
| `StreamHandler`            | 控制台（stdout/stderr） | 最常用，调试阶段                          |
| `FileHandler`              | 文件                    | 简单直接，但不自动轮转                    |
| `RotatingFileHandler`      | 文件                    | 按文件大小轮转（老文件保留）              |
| `TimedRotatingFileHandler` | 文件                    | 按时间（天/小时）轮转                     |
| `NullHandler`              | 不输出任何东西          | 避免“无 handler 警告”                     |
| `SocketHandler`            | TCP socket              | 发给远程日志服务器                        |
| `DatagramHandler`          | UDP socket              | 轻量网络日志                              |
| `HTTPHandler`              | POST 到 HTTP            | 可配合日志收集服务                        |
| `QueueHandler`             | 多进程安全              | 推送到日志队列（用于 async/log listener） |
| `SMTPHandler`              | 发送邮件                | 通常用于严重错误报警                      |
| `MemoryHandler`            | 内存缓冲                | 达到阈值后 flush 到目标 Handler           |
| `WatchedFileHandler`       | 文件                    | 支持 logrotate 自动重载（Linux）          |


### 写入文件（简单）
```python
file = logging.FileHandler("app.log", encoding="utf-8")
file.setLevel(logging.INFO)
```

### 文件轮转（按大小）
```python
from logging.handlers import RotatingFileHandler

rotating = RotatingFileHandler(
    filename="app.log",
    maxBytes=5*1024*1024,  # 每个文件最大5MB
    backupCount=3,         # 最多保留3个旧日志
    encoding="utf-8"
)
rotating.setLevel(logging.INFO)
```

### 文件轮转（按时间）
```python
from logging.handlers import TimedRotatingFileHandler

timed = TimedRotatingFileHandler(
    filename="app.log",
    when="midnight",       # 每天0点切分
    backupCount=7,         # 保留7天日志
    encoding="utf-8"
)
timed.setLevel(logging.INFO)
```

### 多进程日志：推荐使用 QueueHandler
```python
from logging.handlers import QueueHandler, QueueListener
from multiprocessing import Queue

log_queue = Queue()
queue_handler = QueueHandler(log_queue)

listener = QueueListener(log_queue, timed)
listener.start()

logger.addHandler(queue_handler)
```

### Handler 关键属性
| 属性                    | 说明                              |
| ----------------------- | --------------------------------- |
| `.setLevel()`           | 设置最低处理日志等级（如 `INFO`） |
| `.setFormatter()`       | 绑定一个格式器（Formatter）       |
| `.addFilter()`          | 可绑定一个或多个 Filter           |
| `.flush()` / `.close()` | 写入并关闭流（必要时）            |


### 使用多个 Handler 的例子（日志分级）
```python
logger = logging.getLogger("myapp")

# 控制台显示所有日志
console = logging.StreamHandler()
console.setLevel(logging.DEBUG)

# 文件仅记录错误
file = logging.FileHandler("error.log")
file.setLevel(logging.ERROR)

logger.addHandler(console)
logger.addHandler(file)
logger.setLevel(logging.DEBUG)
```

## Formatter 格式化器
`Formatter` 是 Python logging 模块中负责把日志记录格式化为字符串的组件

### 创建 Formatter 的方式
```python
from logging import Formatter

formatter = Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
```

然后把它绑定到 Handler 上：

```python
handler.setFormatter(formatter)
```

### 常用格式化占位符一览
| 占位符           | 含义             | 示例                |
| ---------------- | ---------------- | ------------------- |
| `%(asctime)s`    | 日志时间戳       | 2025-07-14 10:30:10 |
| `%(levelname)s`  | 日志等级         | INFO、DEBUG         |
| `%(levelno)s`    | 日志等级数字     | 20、40              |
| `%(name)s`       | logger 名称      | app.main            |
| `%(filename)s`   | 当前文件名       | main.py             |
| `%(module)s`     | 模块名（无扩展） | main                |
| `%(funcName)s`   | 函数名           | login_user          |
| `%(lineno)d`     | 行号             | 42                  |
| `%(message)s`    | 用户日志内容     | 登录成功            |
| `%(process)d`    | 进程 ID          | 13324               |
| `%(thread)d`     | 线程 ID          | 140737              |
| `%(threadName)s` | 线程名           | Thread-1            |
| `%(pathname)s`   | 全路径           | /project/app.py     |


### 时间格式自定义：datefmt
```python
Formatter('%(asctime)s - %(message)s', datefmt='%Y-%m-%d %H:%M:%S')
```

默认格式：`2025-07-14 10:30:10,423`

你可以自定义为：

```python
Formatter('%(asctime)s', datefmt='%Y/%m/%d %H:%M:%S')
# 输出: 2025/07/14 10:30:10
```

### 完整 Formatter 示例
```python
formatter = logging.Formatter(
    fmt="%(asctime)s [%(levelname)s] %(name)s.%(funcName)s:%(lineno)d - %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)
```

输出示例：

```plain
2025-07-14 10:30:10 [INFO] app.user.login_user:42 - 用户登录成功
```

### 高级 Formatter：添加自定义字段
如果你希望在日志中输出 `trace_id`，你可以通过 `Filter` 注入：

```python
# 自定义 filter 注入字段
class TraceIdFilter(logging.Filter):
    def filter(self, record):
        record.trace_id = "abc123xyz"
        return True
```

Formatter 可以直接用：

```python
'%(asctime)s [%(trace_id)s] %(message)s'
```

### 自定义Formatter子类
你也可以继承 `logging.Formatter`，覆盖 `format(self, record)` 方法，做完全自定义格式控制：

```python
class MyFormatter(logging.Formatter):
    def format(self, record):
        record.msg = f"[MyPrefix] {record.msg}"
        return super().format(record)
```

或：

```python
import json

class JsonFormatter(logging.Formatter):
    def format(self, record):
        log = {
            "time": self.formatTime(record),
            "level": record.levelname,
            "msg": record.getMessage(),
            "trace_id": getattr(record, "trace_id", "-")
        }
        return json.dumps(log, ensure_ascii=False)
```

### 多个 Handler 用不同 Formatter
```python
console_handler.setFormatter(color_formatter)
file_handler.setFormatter(json_formatter)
```

## 使用配置文件
可以使用 `logging.config` 加载 `.ini`、`.json` 或 `.dict` 格式的配置

### dictConfig 示例：
```python
import logging.config

LOGGING_CONFIG = {
    "version": 1,
    "formatters": {
        "default": {
            "format": "%(asctime)s [%(levelname)s] %(name)s: %(message)s"
        }
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "default",
            "level": "DEBUG"
        },
        "file": {
            "class": "logging.FileHandler",
            "formatter": "default",
            "filename": "logfile.log",
            "level": "INFO"
        }
    },
    "root": {
        "handlers": ["console", "file"],
        "level": "DEBUG"
    }
}

logging.config.dictConfig(LOGGING_CONFIG)
logger = logging.getLogger(__name__)
```

### 日志模块最佳实践
| 场景/项目需求                  | 推荐配置                               |
| ------------------------------ | -------------------------------------- |
| 小项目调试                     | `basicConfig()` 即可                   |
| 中大型项目                     | 自定义 Logger + Handler + Formatter    |
| 多模块项目                     | 每个模块单独用 `__name__` 创建 logger  |
| 日志持久化                     | `FileHandler` 或 `RotatingFileHandler` |
| 日志等级分级存储               | 多个 Handler 分别设置不同级别          |
| 日志格式统一                   | 使用配置文件统一控制格式               |
| 想导入外部日志配置文件（YAML） | 用 `yaml` + `dictConfig`               |

