---
title: 如何正确的计算每 N 分钟记录数
date: 2019-09-29 11:15:57
tags: [MySQL]
categories: 后端
---

## 表结构

|字段|数据类型|
|--|--|
|id|int(11)|
|created_at|datetime|

## 期望结果

|时间段|数量|
|--|--|
|2019-09-29 12:00:00 ~ 2019-09-29 12:05:00|100|
|2019-09-29 12:05:00 ~ 2019-09-29 12:10:00|120|
|2019-09-29 12:10:00 ~ 2019-09-29 12:15:00|131|
|2019-09-29 12:15:00 ~ 2019-09-29 12:20:00|189|
|2019-09-29 12:20:00 ~ 2019-09-29 12:25:00|134|
|2019-09-29 12:25:00 ~ 2019-09-29 12:30:00|155|
|2019-09-29 12:30:00 ~ 2019-09-29 12:35:00|198|
|2019-09-29 12:35:00 ~ 2019-09-29 12:40:00|133|

## SQL语句

~~~mysql
-- 300 为 5 分钟
select
  concat(from_unixtime(unix_timestamp(created_at) - unix_timestamp(created_at) % 300), ' ~ ', from_unixtime(unix_timestamp(created_at) - unix_timestamp(created_at) % 300 + 300)) as per,
  count(1)
from table
group by per;
~~~

如果 created_at 类型为 int，还可以减少一次转换

~~~mysql
-- 300 为 5 分钟
select
  concat(from_unixtime(created_at - created_at % 300), ' ~ ', from_unixtime(created_at - created_at % 300 + 300)) as per,
  count(1)
from table
group by per;
~~~

## 思路

> 时间分片 *MySQL没有相关的时间分片函数，还好可以用取模（%）实现*
