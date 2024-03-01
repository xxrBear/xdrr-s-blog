---
title: "Odoo 判断字段自身是否有值 限制修改"
date: 2023-12-26T11:11:45+08:00
lastmod: 2023-12-26T11:11:45+08:00
author: ["熊大如如"]
tags: # 标签
- "odoo"
description: "判断odoo中的一个字段是否写入有值，如果有值则不可修改，如果没有值则可以修改"
summary: "判断odoo中的一个字段是否写入有值，如果有值则不可修改，如果没有值则可以修改"
weight:
slug: ""
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: false # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
---


### 需求

因为工作需要，需要判断odoo中的一个字段是否写入有值，如果有值则不可修改，如果没有值则可以修改

### 失败的做法

*   假设这个字段是 name

在xml中判断name是否存在，存在则只读

```xml
<field name="name" attrs="{'readonly':[('name','!=',False)]}"/>
```

> 上述做法odoo不生效，name字段有值还是可以修改

### 一个正确的做法

**思路**

因为上述判断自己是否有值的做法无法生效，因此采用曲线救国的做法，新增加一个计算字段 `track_name` 用来追踪 name 值是否存在，如果有返回 '1'，如果没有，返回 '0'

**实际代码**

*   模型

```python
track_name = fields.Char(compute='_compute_name')  # 增加计算字段 track_name

@api.onchange('name')  # 追踪 name 的值变化
def _compute_name(self):
	self.ensure_one()
        for record in self:
            if record.name:
                res = '1'
            else:
                res = '0'
            record.track_name = res

```

*   前端

```xml
<!-- 判断track_name的值是否是False -->
<field name="name" attrs="{'readonly':[('track_name','!=','0')]}"/>
```

完成需求，当`name`字段有值时，`track_name`字段为`True`,此时`name`字段不可修改

> 注意：此处我写的`'0'`是字符串，因为我定义的`track_name`字段是`Char`类型，如果你定义的是布尔类型，那么请您自己尝试可用的方法

