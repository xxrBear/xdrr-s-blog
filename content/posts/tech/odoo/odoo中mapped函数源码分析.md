---
title: "Odoo中mapped函数源码分析"
date: 2024-02-19T15:40:00+08:00
lastmod: 2024-02-19T15:40:00+08:00
author: ["熊大如如"]
tags: 
- "odoo"
description: ""
weight:
slug: ""
summary: "odoo9 mapped函数源码分析"
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

当我们使用`mapped`函数时，我们会给函数传一个odoo模型的字段参数字符串，例如：

```python
so = self.env['sale.order'].sudo().search([(id, '=', 212574)])

so.mapped('name')
Out[1]: [u'SO155168440244263003']  # 输出

so.mapped('partner_id')
Out[2]: res.partner(778648,)
```

*   当传入的字段属性不是关联外键时，它会输出一个字段值列表，当传入的字段属性是外键时，它会返回关联的外键对象集合

或者，我们会传入多个字符串参数以点号分开，例如

```python
so.mapped('partner_id.name')
Out[3]: [u'\u978d\u94a2\u7eff\u8272\u8d44\u6e90\u79d1\u6280\u6709\u9650\u516c\u53f8']

so.mapped('order_line.order_id')
Out[4]: sale.order(212574,)
```

*   与上面一样，传入关联字段的属性也遵循这个原则

那么你有没有想过odoo处理这种数据的原理是什么呢？？？

### 2.源码及版本说明

> 版本：
>
> odoo9
>
> python2.7

贴出源码

```python
def _mapped_func(self, func):
    """ 
    将函数“func”应用于“self”中的所有记录，并将结果作为列表或记录集返回（如果“func”返回记录集）。
    """
    if self:
        vals = [func(rec) for rec in self]
        if isinstance(vals[0], BaseModel):
            # return the union of all recordsets in O(n)
            ids = set(itertools.chain(*[rec._ids for rec in vals]))
            return vals[0].browse(ids)
        return vals
    else:
        vals = func(self)
        return vals if isinstance(vals, BaseModel) else []

def mapped(self, func):
    """
    将“func”应用于“self”中的所有记录，并将结果作为列表或记录集返回（如果“func”返回记录集）。在后一种情况下，返回的记			    录集的顺序是任意的。

    :param func: 一个函数或一系列用点分隔的字段名
    """
    if isinstance(func, basestring):
        recs = self
        for name in func.split('.'):
            recs = recs._mapped_func(operator.itemgetter(name))
        return recs
    else:
        return self._mapped_func(func)
```

### 3.itemgetter函数的作用

```python
from operator import itemgetter

f = itemgetter(2)
此时，调用f(r)等于调用r(2)
```

**此时，调用f(r)等于调用r(2)**

> **注意：itemgetter是调用** **`__gititem__` 方法来获取属性，所以对函数对象无法使用**

### 4.代码解析

*   当参数是字符串时

a.对参数字符串进行循环处理，然后使用`_mapped_func`函数处理参数

b.在`_mapped_func`函数中，列表推导式中对resc对象也就是self对象循环调用 func(rec)

假设你写的是`mapped('name')`，那么在此时，就会遍历self对象，获取name属性，而不是func('name')，前面已经说过，这是从\_\_getitem\_\_方法中取值。

c.当你写的参数取出来的值是BaseModel的实例时，遍历self对象，最后获取所有的实例对象并返回，这就是为什么写关联字段的值，得到关联对象集合的原因。

*   当参数不是字符串的时候

a.直接对func参数，调用`_mapped_func`方法

b.对self对象进行循环处理，最后还是返回列表或者对象集

*   当非实例对象调用时

非实例对象调用时，例如 类对象

> 参数可以直接对类对象进行操作

举个例子：

```python
so = self.env['sale.order'].sudo()

so.mapped(lambda self: self.search([('id', =, 92249)]))

Out[5]: sale.order(92249,)
```

*   当self是空时

当self是空值时，等同于 类对象直接调用 mapped 方法

与上面的例子重合
