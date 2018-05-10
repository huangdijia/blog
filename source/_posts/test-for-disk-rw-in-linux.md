---
title: Linux下测试硬盘读写速度
date: 2012-3-16 13:40:15
tags: [linux, dd]
categories: 后端
---

time有计时作用
dd用于复制，从if读出，写到of
`if=/dev/zero`不产生IO，因此可以用来测试纯写速度。
同理`of=/dev/null`不产生IO，可以用来测试纯读速度。
bs是每次读或写的大小，即一个块的大小，count是读写块的数量。
<!--more-->
### 1.测/目录所在磁盘的纯写速度：
~~~bash
time dd if=/dev/zero bs=1024 count=1000000 of=/1Gb.file
~~~
### 2.测/目录所在磁盘的纯读速度：

~~~bash
dd if=/kvm/ftp/other/1Gb.file bs=64k |dd of=/dev/null
~~~
### 3.测读写速度(这是什么)：

~~~bash
dd if=/vat/test of=/oradata/test1 bs=64k
~~~

> 理论上复制量越大测试越准确。