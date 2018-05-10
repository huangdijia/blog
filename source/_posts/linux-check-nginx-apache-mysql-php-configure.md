---
title: Linux下查看nginx，apache，mysql，php的编译参数
date: 2012-12-13 13:32:02
tags: [nginx, php, mysql, apache, configure, linux]
categories: 后端
---

在升级软件版本的时候，如果没有文档，那是相当痛苦的事情，编译参数是什么？

<!--more-->

1、nginx编译参数：

~~~bash
/usr/local/nginx/sbin/nginx -V
~~~

2、apache编译参数：

~~~bash
cat /usr/local/apache/build/config.nice
~~~

3、php编译参数：

~~~bash
/usr/local/php/bin/php -i |grep configure
~~~

4、mysql编译参数：

~~~bash
cat /usr/local/mysql/bin/mysqlbug|grep configure
~~~
