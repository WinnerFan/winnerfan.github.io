---
layout: post
title: MySQL Paging
tags: MySQL
---
## 分页背景
MySQL的分页查询在应用业务十分常见。针对于不同的使用环境，选用的分页语句不尽相同。本文针对不同的应用场景整理了查询语句，执行并EXPLAIN查看性能。
## 分页语句
### LIMIT m,n语句
```
SELECT * FROM table ORDER BY id DESC LIMIT 100,10;
```
**少量数据**时，可以使用。但是，这种分页查询方式会从数据库第一条记录开始扫描，所以越往后，查询的数据越多，查询速度越慢。例如表中共1千万条数据时，limit 8000000,10每次查询需要扫描为8000010行，耗时15.75秒，效率十分低下。

### LIMIT m语句
```
SELECT * FROM table WHERE id > 100 ORDER BY id ASC LIMIT 10;//下一页
SELECT * FROM table WHERE id < 100 ORDER BY id DESC LIMIT 10;//上一页
```
**大量数据**时，如果索引查询的语句中只包含了索引列（覆盖索引），则查询速度大大加快。这是因为利用索引查找有优化算法；数据就在查询索引上面，不用再去找相关的数据地址；另外MySQL中也有相关的索引缓存，在并发高的时候利用缓存效果更好。当表中共1千万条数据，id > 8000000 limit 100扫描行数为3844842行，耗时为0.00秒。

### BETWEEN AND语句
```
SELECT * FROM table WHERE id BETWEEN 8000000 AND 8000009;
```
**大量数据**时，查询id连续的时，这种方法仅仅需要扫描10条数据，耗时0.00秒。

### IN语句
```
SELECT * FROM table WHERE id IN (800,8000,80000,800000,8000000);
```
**大量数据**时，查询id不是连续的情况，这种方法只需要扫描5条数据，耗时0.00秒。
