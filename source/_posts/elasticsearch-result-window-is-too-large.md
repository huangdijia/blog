---
title: ES报错Result window is too large问题处理
date: 2018-06-26 12:31:46
tags: [es, elasticsearch]
categories: 数据库
---

我在使用Elasticsearch进行search查询的过程中，出现了Result window is too large问题。

> Result window is too large, from + size must be less than or equal to: [10000] but was [43155]. See the scroll api for a more efficient way to request large data sets. This limit can be set by changing the [index.max_result_window] index level setting.

<!--more-->

从上面的报错信息，可以看到ES提示我结果窗口太大了，目前最大值为`10000`，而我却要求给我`43155`。并且在后面也提到了要求我修改`index.max_result_window`参数来增大结果窗口大小。

修改方法，命令如下：

~~~bash
curl -XPUT -H "Content-Type: application/json" http://192.168.8.82:9200/indexName/_settings -d '{"index" : {"max_result_window" : 100000000}}'
~~~
