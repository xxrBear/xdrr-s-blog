---
title: "PostgreSQL 联合索引生效条件"
date: 2025-05-15T21:25:05+08:00
lastmod: 2025-05-15T21:25:05+08:00
author: ["熊大如如"]
tags: # 标签
  - "PostgreSQL"
description: ""
weight:
slug: ""
summary: "PostgreSQL 联合索引生效条件"
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
最近面试的时候，总会遇到一个问题

> 在 PostgreSQL 中，联合索引在什么条件下会生效？
>

特此记录~

## 前置信息
数据库版本

+ PostgreSQL 14.13, compiled by Visual C++ build 1941, 64-bit

建表语句

```sql
CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    city VARCHAR(50),
    name VARCHAR(50),
    age INT
);

CREATE INDEX idx_city_name_age ON people(city, name, age);

-- 插入数据
INSERT INTO people (city, name, age) VALUES
('Beijing', 'Tom', 18),
('Beijing', 'Tom', 20),
('Beijing', 'Jerry', 18),
('Shanghai', 'Jerry', 22),
('Shanghai', 'Alice', 25),
('Guangzhou', 'Bob', 30);
```

## 查看索引是否命中
```sql
EXPLAIN ANALYZE SELECT * FROM people WHERE city = 'Beijing' AND name = 'Tom' AND age = 20;
```

结果

```sql
Index Scan using idx_city_name_age on people  (cost=0.15..8.17 rows=1 width=244) (actual time=0.012..0.013 rows=1 loops=1)
  Index Cond: (((city)::text = 'Beijing'::text) AND ((name)::text = 'Tom'::text) AND (age = 20))
Planning Time: 0.074 ms
Execution Time: 0.022 ms
```

我们可以清楚的看到索引命中了，那么改变一下 WHERE 条件呢，例如：

```sql
EXPLAIN ANALYZE SELECT * FROM people WHERE city = 'Beijing';
EXPLAIN ANALYZE SELECT * FROM people WHERE name = 'Tom';
EXPLAIN ANALYZE SELECT * FROM people WHERE age = 20;
EXPLAIN ANALYZE SELECT * FROM people WHERE name = 'Tom' and age = 20;
EXPLAIN ANALYZE SELECT * FROM people WHERE city = 'Beijing' and name = 'Tom';
EXPLAIN ANALYZE SELECT * FROM people WHERE city = 'Beijing' and age = 20;
```

分别执行一下，查看结果

```sql
Bitmap Heap Scan on people  (cost=4.16..9.50 rows=2 width=244) (actual time=0.014..0.014 rows=3 loops=1)
  Recheck Cond: ((city)::text = 'Beijing'::text)
  Heap Blocks: exact=1
  ->  Bitmap Index Scan on idx_city_name_age  (cost=0.00..4.16 rows=2 width=0) (actual time=0.011..0.011 rows=3 loops=1)
        Index Cond: ((city)::text = 'Beijing'::text)
Planning Time: 0.056 ms
Execution Time: 0.031 ms

-------------------------------------------------------------------------------
Seq Scan on people  (cost=0.00..13.75 rows=2 width=244) (actual time=0.009..0.010 rows=2 loops=1)
  Filter: ((name)::text = 'Tom'::text)
  Rows Removed by Filter: 4
Planning Time: 0.060 ms
Execution Time: 0.020 ms

-------------------------------------------------------------------------------
Seq Scan on people  (cost=0.00..13.75 rows=2 width=244) (actual time=0.008..0.009 rows=1 loops=1)
  Filter: (age = 20)
  Rows Removed by Filter: 5
Planning Time: 0.054 ms
Execution Time: 0.017 ms

-------------------------------------------------------------------------------
Seq Scan on people  (cost=0.00..14.50 rows=1 width=244) (actual time=0.011..0.012 rows=1 loops=1)
  Filter: (((name)::text = 'Tom'::text) AND (age = 20))
  Rows Removed by Filter: 5
Planning Time: 0.056 ms
Execution Time: 0.020 ms

-------------------------------------------------------------------------------
Index Scan using idx_city_name_age on people  (cost=0.15..8.17 rows=1 width=244) (actual time=0.015..0.016 rows=2 loops=1)
  Index Cond: (((city)::text = 'Beijing'::text) AND ((name)::text = 'Tom'::text))
Planning Time: 0.057 ms
Execution Time: 0.026 ms

-------------------------------------------------------------------------------
Index Scan using idx_city_name_age on people  (cost=0.15..8.18 rows=1 width=244) (actual time=0.018..0.019 rows=1 loops=1)
  Index Cond: (((city)::text = 'Beijing'::text) AND (age = 20))
Planning Time: 0.062 ms
Execution Time: 0.031 ms
```

查看结果，我们发现，WHERE条件后面带上 `city`字段的 SQL 语句全部走了索引，其余字段全表扫描。

看到这儿，可能可以得出结论，PG 数据库联合索引遵循最左匹配原则，只有最左边的字段存在才能命中索引。

这里才几条数据，让我们增加 people 表中的数据

```sql
INSERT INTO people (city, name, age)
SELECT
    -- 随机分配 3 个城市
    CASE (random()*3)::int
        WHEN 0 THEN 'Beijing'
        WHEN 1 THEN 'Shanghai'
        ELSE 'Guangzhou'
    END,
    -- 随机分配 10 个名字
    CASE (random()*10)::int
        WHEN 0 THEN 'Tom'
        WHEN 1 THEN 'Jerry'
        WHEN 2 THEN 'Alice'
        WHEN 3 THEN 'Bob'
        WHEN 4 THEN 'David'
        WHEN 5 THEN 'Eva'
        WHEN 6 THEN 'John'
        WHEN 7 THEN 'Lily'
        WHEN 8 THEN 'Lucy'
        ELSE 'Mike'
    END,
    (random()*100)::int  -- 年龄 0~100
FROM generate_series(1, 100000);
```

这行 SQL 语句为 people 表增加了十万条数据，让我们再试试没有 `city`字段的查询语句

```sql
EXPLAIN ANALYZE SELECT * FROM people WHERE age = 18 and name = 'David';
```

执行结果

```sql
Bitmap Heap Scan on people  (cost=1556.38..1836.90 rows=107 width=22) (actual time=0.485..0.532 rows=80 loops=1)
  Recheck Cond: (((name)::text = 'David'::text) AND (age = 18))
  Heap Blocks: exact=78
  ->  Bitmap Index Scan on idx_city_name_age  (cost=0.00..1556.35 rows=107 width=0) (actual time=0.477..0.477 rows=80 loops=1)
        Index Cond: (((name)::text = 'David'::text) AND (age = 18))
Planning Time: 0.856 ms
Execution Time: 0.549 ms
```

我们看到没有 city 字段，还是走了索引。但是似乎这种方式不是高效的方式，PG 数据库综合考虑还是走了索引，别的情况下可能不会走索引。

但是单独的非最左索引字段肯定不走索引，例如：

```sql
EXPLAIN ANALYZE SELECT * FROM people WHERE age = 18;
EXPLAIN ANALYZE SELECT * FROM people WHERE name = 'David';
```

结果

```sql
Seq Scan on people  (cost=0.00..1887.08 rows=1050 width=22) (actual time=0.009..6.824 rows=1039 loops=1)
  Filter: (age = 18)
  Rows Removed by Filter: 98967
Planning Time: 0.072 ms
Execution Time: 6.857 ms

----------------------------------------------------------------
Seq Scan on people  (cost=0.00..1887.08 rows=10204 width=22) (actual time=0.014..6.997 rows=10041 loops=1)
  Filter: ((name)::text = 'David'::text)
  Rows Removed by Filter: 89965
Planning Time: 0.057 ms
Execution Time: 7.207 ms
```

全表扫描，没有走索引

## 总结
总的来说，Postgres 数据库联合索引生效条件遵循最左匹配原则。

在本例中，也就是说 `city`字段存在于 `where`条件的后面，才能高效的使用索引，如果没有 `city`字段，可能也会命中索引，但是是不高效的，当三个联合索引字段都存在时，这时是最高效的查询语句。

另外，查询字段在 WHERE 条件后面的顺序是不妨碍索引是否命中的，PG 优化器会识别优化。

你对 Postgre 数据库的联合索引有了解吗？你是否知道何种情况下 PG 数据库会高效的命中索引，可以留下你的评论，我们一起讨论。

如有错误，请指正~

