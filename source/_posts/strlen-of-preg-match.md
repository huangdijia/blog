---
title: preg_match正则匹配的字符串长度问题
date: 2012-7-17 13:11:42
tags: [php, 正则, regex]
categories: 后端
---

项目中，用`preg_match`正则提取目标内容，死活有问题，代码测得死去活来。
后来发现`pcre.backtrack_limit`的值默认只设了`100000`。
<!--more-->
解决办法：
~~~php
ini_set(‘pcre.backtrack_limit’, 999999999);
~~~
注：这个参数在php 5.2.0版本之后可用。
另外说说关于：`pcre.recursion_limit`
`pcre.recursion_limit`是`PCRE`的递归限制，这个项如果设很大的值，会消耗所有进程的可用堆栈，最后导致PHP崩溃。
也可以通过修改配置来限制：
~~~php
ini_set(‘pcre.recursion_limit’, 99999);
~~~
实际项目应用中，最好也对内存进行限定设置：
~~~php
ini_set(‘memory_limit’, ’64M’); 
~~~
这样就比较稳妥妥嘎。