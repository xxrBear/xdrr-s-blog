---
title: "《健壮的Python》"
date: 2025-04-09T22:24:53+08:00
lastmod: 2025-04-09T22:24:53+08:00
author: ["熊大如如"]
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Robust Python"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202504101054081.jpg" # 文章的图片
---

## 简介

这是一本教你如何写出健壮 Python 代码的书，里面包含了多种编程技巧和编程工具。

本书可以分为四个部分，第一部分主要介绍了什么是 Python 的类型注解，Python 类型注解是 Python3.5 引入的 Python 新特性，可以帮助 Python 代码规避动态语言的一些劣势，例如执行时才能发现函数参数类型错误,帮助我们提早发现代码的错误，还有检查 Python 类型的类型检查器。

第二部分介绍如何定义你自己的数据类型，例如枚举类和数据类，Enum 和 @dataclass 的用法，接口、子类型、协议等。但是看了理解不是很深，可能是自己太菜了，也可能是
这些用法似乎是 Python 库开发者才会经常用到的技巧 😂

第三部分是教你如何写出可扩展性的 Python 代码，也就是在 Python 中常见的一些设计模式和开发原则，例如经典的开放-封闭原则，对代码的修改封闭，扩展开放。代码的可组合性，使用组合而不是使用继承。事件驱动架构的一些实践，用来解决代码耦合问题和提高任务处理性能。还有对代码依赖关系进行可视化等等优化方法。

第四部分，教你怎么完整的对代码进行各种测试，没看！！！

总结一下，本书比较适合一些具有开发经验的 Python 程序员，跟《Python 工匠》它介绍了一系列写出健壮的 Python 代码的最佳实践，值得一看。
