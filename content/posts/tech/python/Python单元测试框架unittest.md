---
title: "Python单元测试库unittest"
date: 2024-04-25T11:44:00+08:00
lastmod: 2024-04-25T11:44:00+08:00
author: ["熊大如如"]
keywords:
  - "python"
categories: # 分类
  -  # 在这儿写分类
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "python单元测试模块unittest的快速入门"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---

## 1.什么是单元测试

单元测试是软件测试的一种方法，是测试最小的可测试单元，通常是函数或一个类。

## 2.为什么要单元测试

可以帮我们尽早发现问题，定位问题，解决问题。

## 3.快速开始 unittest

- 一个函数

```python
def add(x, y):
    return x+y
```

- 一个测试类

```python
import unittest

from demo import add

class TestMain(unittest.TestCase):
    def test_add(self):
        result = add(1, 2)
        self.assertEqual(result, 3)


if __name__ == '__main__':
    unittest.main()
```
