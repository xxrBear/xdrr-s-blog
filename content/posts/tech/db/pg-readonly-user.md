---
title: "PostgreSQL 创建只读账户"
date: 2025-06-12T10:12:45+08:00
lastmod: 2025-06-12T10:12:45+08:00
author: ["熊大如如"]
tags: # 标签
  - "postgresql"
description: ""
weight:
slug: ""
summary: "pg数据库如何创建只读账户"
draft: false # 是否为草稿
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "https://cdn.jsdelivr.net/gh/xxrBear/image//Hugo/202505152126346.png"  # 文章的图片
---
## 一、创建用户
```sql
CREATE USER readonly WITH PASSWORD 'readonly';
```

## 二、授予只读权限
你可以对一个数据库下的所有表授予只读权限。下面是授予权限的通用步骤：

### 1. 授予连接权限
```sql
GRANT CONNECT ON DATABASE mydb TO readonly;
```

### 2. 授予 schema 的使用权限
```sql
GRANT USAGE ON SCHEMA public TO readonly;
```

> 如果你使用的是其他 schema，请替换 `public`
>

### 3. 授予所有已有表的 SELECT 权限
```sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
```

### 4. 授予未来表的 SELECT 权限（防止以后新增的表无法访问）
```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO readonly;
```

## 三、禁止写操作
虽然只授予了 SELECT 权限，但你可能仍想防止用户误操作，比如创建表、更新表结构，可以拒绝这些权限：

```sql
-- 禁止创建表等 DDL 操作
REVOKE CREATE ON SCHEMA public FROM readonly;
```

## 四、验证权限
以该用户连接数据库后执行以下命令测试权限：

```sql
-- 测试查询（应该成功）
SELECT * FROM some_table;

-- 测试写入（应该失败）
INSERT INTO some_table (col) VALUES ('test');
```

