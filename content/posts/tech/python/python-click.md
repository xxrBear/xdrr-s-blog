---
title: "Python Click"
date: 2025-07-16T16:36:52+08:00
lastmod: 2025-07-16T16:36:52+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
summary: "CLI工具 click 简单入门"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://raw.githubusercontent.com/xxrBear/image/master/blog/click-logo%20(1).jpg"  # 文章的图片
---
## 简介
`Click` 是 Python 中用于构建命令行界面的强大库，设计目标是：用尽可能少的代码，构建可组合的命令行界面

## 入门
```python
import click

@click.command()
@click.option('--count', default=1, help='重复次数')
@click.argument('name')
def hello(count, name):
    for _ in range(count):
        click.echo(f"Hello, {name}!")

if __name__ == '__main__':
    hello()
```

运行

```python
$ python app.py --count=3 Alice
Hello, Alice!
Hello, Alice!
Hello, Alice!
```

## 核心组件
###  注册命令
```python
@click.command()
```

###  位置参数
```python
@click.argument('filename')
调用时必须写
$ python app.py myfile.txt
```

### 可选参数
```python
@click.option('--debug', is_flag=True, help='开启调试模式')
@click.option('--level', type=int, default=3)
```

### 输出函数
```python
click.echo()
```

### 类型系统
```python
@click.argument('path', type=click.Path(exists=True))
```

### 创建命令组
```python
@click.group()
def cli():
    pass

@cli.command()
def init():
    click.echo("初始化")

@cli.command()
def start():
    click.echo("启动")

if __name__ == '__main__':
    cli()
```

执行

```python
$ python app.py init
初始化
```

### 提示
```python
@click.option('--username', prompt='用户名')
@click.option('--password', prompt=True, hide_input=True)
```

### 回调函数
在参数解析后执行特定函数

```python
def validate(ctx, param, value):
    if value < 0:
        raise click.BadParameter('必须为非负数')
    return value

@click.option('--num', callback=validate, type=int)
```

### 命令别名
```python
@click.command(name='run', help='执行')
```

### 自定义帮助输出
```python
@click.group(context_settings=dict(help_option_names=['-h', '--help']))
```

## 实用场景
### 多值选项
```python
@click.option('--path', multiple=True)
```

调用

```python
$ python app.py --path a --path b
```

### 布尔值选项
```python
@click.option('--verbose/--no-verbose', default=True)
```

### 环境变量选项
```python
@click.option('--config', envvar='APP_CONFIG')
```

## 实战项目结构
下面是一个实战级的 Python CLI 工具项目示例，基于 Click 实现，模拟一个日常常见的命令行工具

### 项目目标
我们创建一个名为 `tasker` 的命令行工具，用来：

+ 管理待办事项（任务列表）
+ 支持子命令：`add`, `list`, `done`, `remove`
+ 本地保存任务（使用 JSON 文件作为数据库）
+ 可选支持 `--verbose` 模式

### 项目结构
```plain
tasker/
├── __main__.py      # 入口点
├── cli.py           # 命令组定义
├── commands/
│   ├── add.py       # 添加任务
│   ├── list.py      # 列出任务
│   ├── done.py      # 标记完成
│   └── remove.py    # 删除任务
├── storage.py       # JSON 文件存储逻辑
├── model.py         # 任务数据结构
├── config.py        # 配置管理（如路径）
└── task_db.json     # 数据文件（运行时自动生成）
```

### 安装入口（可选）
在 `pyproject.toml` 或 `setup.py` 中添加：

```toml
[project.scripts]
tasker = "tasker.__main__:cli"
```

### 代码示例
`__main__.py`

```python
from tasker.cli import cli

if __name__ == "__main__":
    cli()
```

 `cli.py`

```python
import click
from tasker.commands import add, list, done, remove

@click.group()
@click.option('--verbose', is_flag=True, help="显示详细信息")
@click.pass_context
def cli(ctx, verbose):
    ctx.ensure_object(dict)
    ctx.obj['verbose'] = verbose

cli.add_command(add.command)
cli.add_command(list.command)
cli.add_command(done.command)
cli.add_command(remove.command)
```

`commands/add.py`

```python
import click
from tasker.storage import load_tasks, save_tasks

@click.command()
@click.argument('title')
@click.pass_context
def command(ctx, title):
    tasks = load_tasks()
    tasks.append({'title': title, 'done': False})
    save_tasks(tasks)
    click.echo(f"✅ 添加任务: {title}")
```

 `commands/list.py`

```python
import click
from tasker.storage import load_tasks

@click.command()
@click.pass_context
def command(ctx):
    tasks = load_tasks()
    for i, task in enumerate(tasks, 1):
        status = "✔" if task['done'] else "✘"
        click.echo(f"{i}. [{status}] {task['title']}")
```

`commands/done.py`

```python
import click
from tasker.storage import load_tasks, save_tasks

@click.command()
@click.argument('index', type=int)
@click.pass_context
def command(ctx, index):
    tasks = load_tasks()
    if 0 < index <= len(tasks):
        tasks[index - 1]['done'] = True
        save_tasks(tasks)
        click.echo(f"🎉 任务完成: {tasks[index - 1]['title']}")
    else:
        click.echo("❌ 无效的任务编号")
```

`commands/remove.py`

```python
import click
from tasker.storage import load_tasks, save_tasks

@click.command()
@click.argument('index', type=int)
@click.pass_context
def command(ctx, index):
    tasks = load_tasks()
    if 0 < index <= len(tasks):
        removed = tasks.pop(index - 1)
        save_tasks(tasks)
        click.echo(f"🗑 删除任务: {removed['title']}")
    else:
        click.echo("❌ 无效的任务编号")
```

`storage.py`

```python
import os
import json

DB_PATH = os.path.expanduser("~/.tasker_db.json")

def load_tasks():
    if not os.path.exists(DB_PATH):
        return []
    with open(DB_PATH, "r") as f:
        return json.load(f)

def save_tasks(tasks):
    with open(DB_PATH, "w") as f:
        json.dump(tasks, f, indent=2)
```

### 示例运行效果
```bash
$ tasker add "写 Click 教程"
✅ 添加任务: 写 Click 教程

$ tasker list
1. [✘] 写 Click 教程

$ tasker done 1
🎉 任务完成: 写 Click 教程

$ tasker list
1. [✔] 写 Click 教程

$ tasker remove 1
🗑 删除任务: 写 Click 教程
```

