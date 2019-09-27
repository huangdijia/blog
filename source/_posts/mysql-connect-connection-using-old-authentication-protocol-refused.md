---
title: MySQL Connection using old authentication protocol refused
date: 2018-05-17 18:24:40
tags: [mysql, authentication]
categories: 数据库
---

有一台mysql升级到5.6版本，结果连接一些低版本的mysql服务器报出如下异常：

> Warning: mysql_connect(): Connection using old (pre-4.1.1) authentication protocol refused (client option 'secure_auth' enabled)

异常原因在于服务器端的密码管理协议陈旧，使用的是旧有的用户密码格式存储；但是客户端升级之后采用了新的密码格式。mysql5.6版本遇到这种不一致的情况就会拒绝连接。

<!--more-->

详见mysql手册 `Server Command Options` 一节中 `--secure-auth` 选项的[说明](http://dev.mysql.com/doc/refman/5.6/en/server-options.html#option_mysqld_secure-auth)

解决方法有如下两种：

> 1、服务器端升级启用 `secure_auth` 选项；
> 2、客户端连接时off掉 `secure_auth` ，即连接时加上 `--secure_auth=off`，如：`mysql -p10.51.1.11 -P3308 -uroot --secure_auth=off`
> 对于方法二，使用在程序做相应 mysql 配置即可，以php为例，在 `php.ini` 中设置 `secure_auth=off`
