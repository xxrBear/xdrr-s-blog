---
title: "ODOO 慢查询优化"
date: 2024-04-22T18:00:24+08:00
lastmod: 2024-04-22T18:00:24+08:00
author: ["熊大如如"]
keywords: 
- 
categories: # 分类
- # 在这儿写分类
tags: # 标签
- "odoo"
description: ""
weight:
slug: ""
summary: "odoo如何进行慢查询优化"
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

### odoo打开显示慢查询日志
```shell
python odoo.py shell --log-level=debug_sql
```

这会将orm查询输出为SQL查询，并显示慢查询时间