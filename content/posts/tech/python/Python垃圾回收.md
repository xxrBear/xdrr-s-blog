---
title: "Python垃圾回收机制"
date: 2023-02-08T17:13:42+08:00
lastmod: 2023-02-08T17:13:42+08:00
author: ["熊大如如"]
tags: # 标签
  - "Python"
description: ""
weight:
slug: ""
summary: "简单介绍了Python的垃圾回收机制和Python缓存机制"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: false # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://raw.githubusercontent.com/xxrBear/image/master/icons8-python-500.png"
---

### Python对象的头部信息
熟悉Python的都知道一个概念，一切皆对象。python对象包含了两个头部信息：一个是类型标志符，另一个是引用计数器。

想要了解后者，需要继续往下看。

### Python对象的垃圾收集
在内部，Python是这样来实现垃圾回收的。它在每一个对象中保留了一个计数器，计数器记录着当前对象被引用的次数。

例如：
```angular2html
a = [99, 99, 990]
```
此时对象 [99, 99, 990] 的引用次数会加一，注意：此处对象指的是列表，而不是变量a, a只是列表对象的一个引用。

此时如果变量a的引用不指向列表对象，那么列表对象的引用计数器变为零，就会导致它的空间被回收。

### Python对象缓存机制
Python内部缓存并复用了小的整数和小的字符串，因此，对于小整数1，也许并不一定会被空间回收。

例子：
```ipython
In [1]: a = 1

In [2]: b = 1

In [3]: a is b
Out[3]: True
```