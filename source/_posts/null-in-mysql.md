---
title: 未知的恐惧——NULL
date: 2013-4-27 18:16:32
tags: [mysql, 'null']
categories: 数据库
---

~~~sql
SELECT
    NULL = 0,
    NULL = 12345,
    NULL <> 12345,
    NULL + 12345,
    NULL || 'abc',
    NULL = NULL ,
    NULL <> NULL ,
    NULL AND TRUE ,
    NULL AND FALSE ,
    NULL OR FALSE ,
    NULL OR TRUE ,
    NOT (NULL);
~~~

如果这是一道面试题，估计不知道有多少程序员甚至是DBA会阵亡。
<!--more-->
正确的答案是什么？（为了加深印象，建议复制SQL到mysql里去执行，看一下）

下面跟大家分析一下原因：

![图片](/images/null-in-mysql.png)

那么在应用中如何避免NULL带来的一些困扰呢？

> - 把NULL当成一个特殊值，不等于空、0、FALSE，使用IS NULL/IS NOT NULL去检测
> - 声明NOT NULL列，给于默认值
