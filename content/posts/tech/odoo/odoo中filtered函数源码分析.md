---
title: "Odoo中filtered函数源码分析"
date: 2024-02-20T17:51:47+08:00
lastmod: 2024-02-20T17:51:47+08:00
author: ["熊大如如"]
keywords: 
- 
categories: # 分类
- # 在这儿写分类
tags: # 标签
-
description: ""
weight:
slug: ""
summary: ""
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: ""  # 文章的图片
---

### 1.问题描述

当我们使用`filtered`函数时，我们会传入一个字符串参数或者一个函数

*   当传入一个字符串的时候，例子：

```python
so = self.env['sale.order'].sudo().search([('id', 'in', (148199, 251969))])

Out[1]: sale.order(148199, 251969)

In [2]: for s in so:
    ...:     print s.city_name
    ...:
False
邵阳市

In [3]: print so.filtered('city_name').city_name
邵阳市
```

两条记录，一条`city_name`有值，一个没有

当传入参数`city_name`时，会过滤掉`city_name`为空的记录

*   当传入一个函数的时候，例子：

```python
so.filtered(lambda self: self.city_name == '邵阳市')

In [11]: so.filtered(lambda self: self.city_name == '邵阳市')
Out[11]: sale.order(251969,)
```

返回执行函数返回值为`True`的记录

**那么这是为什么呢？？？**

### 2.源码及版本说明

> python2.7
>
> odoo9

贴出源码

```python
def filtered(self, func):
"""
            选择“self”中的记录，使得“func(rec)”为真，并且
            将它们作为记录集返回。

            :param func：函数或点分隔的字段名称序列
"""
    if isinstance(func, basestring):
        name = func
        func = lambda rec: filter(None, rec.mapped(name))
    return self.browse([rec.id for rec in self if func(rec)])
```

### 3.代码分析

#### filter函数的使用

这里用的是`filter`函数第一个参数是`None`的情况，这种情况下，会过滤掉序列中的空值

例子：

```python
In [1]: num_list = [2, 4, 0]
   ...: filter(None, num_list)

Out[1]: [2, 4]
```

#### 当参数为字符串时

假设参数为`city_name`，会把`city_name`传入`mapped`函数中，这样最后会找出所有`self`实例的`city_name`值，然后过滤掉`city_name`为空的记录

#### 当参数是函数时

这里直接对所有实例循环使用`func`方法，过滤空值

### 4.总结

`filtered`方法就是对`mapped`方法的封装，去除了不符合条件的值
