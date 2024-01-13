---
title: "Python函数"
date: 2024-01-10T22:06:25+08:00
lastmod: 2024-01-10T22:06:25+08:00
author: ["熊大如如"]
keywords: 
- 
categories: # 分类
- # 在这儿写分类
tags: # 标签
  - "python"
description: ""
weight:
slug: ""
summary: "Python函数的相关内容"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: false # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---

### keyword-only 函数
Python3有一种只有关键字参数，只能传递关键字参数
+ 语法
```python
def kwonly(*, name='spam', age=1):
    ...

kwonly('eggs', 18)

TypeError: kwonly() takes 0 positional arguments but 2 were given
```
> 传入非关键字参数会报错

+ 正确的语法
```python
kwonly(name='eggs', age=18)
```