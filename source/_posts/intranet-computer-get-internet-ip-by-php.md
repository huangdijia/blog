---
title: 内网机器的获取公网IP的方法
date: 2012-12-13 13:09:27
tags: [php, getclientip]
categories: 后端
---

~~~php
function getClientIp(){  
    $socket = socket_create(AF_INET, SOCK_STREAM, 6);  
    $ret = socket_connect($socket,'ns1.dnspod.net',6666);  
    $buf = socket_read($socket, 16);  
    socket_close($socket);  
    return $buf;
}
~~~

缺点：依赖第三方，效率与网络状况有关。
