---
title: "Python列表"
date: 2023-02-08T20:12:02+08:00
lastmod: 2023-02-08T20:12:02+08:00
author: ["熊大如如"]
keywords:
  - "python"
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "python列表的常用方法"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: false # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image/icons8-python-500.png"
---

### 1.什么是 Python 列表

Python 内置的数据格式，可变的、可迭代的对象。广泛应用在 Python 程序中。

### 2.列表的常用方法

- 向后追加列表数据

```
name_list = []

name_list.append('spam')
```

- 指定索引位置增加数据

```
name_list.insert(0, 'bob')
```

- 删除列表最后一个数据

```python
name = name_list.pop()
```

- 删除列表第一个数据

```python
name = name_list.pop(0)
```

### 3.列表实现先进先出队列

```python
class Name:
    def __init__(self):
        self.name_list = []
    def add_name(self, name):
        self.name_list.append(name)
    def get_name(self):
        return self.name_list.pop(0)
```
