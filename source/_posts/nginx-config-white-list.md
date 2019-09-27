---
title: nginx安全配置之白名单设置
date: 2014-1-6 13:16:46
tags: [nginx, webserver, php]
categories: 后端
---

> 网站安全，防不胜防

你永远不知道黑客利用什么漏洞对你的网站进行攻击，最常见的手段是利用漏洞在你的网站留个后门，喜欢的时候来逛逛，比如玩你的服务器上传一个x.php，直接访问：

~~~bash
http://www.example.com/a/b/c/d/a.php
~~~

然后就可以为所欲为了，那么有没有办法让黑客即使利用漏洞上传了php代码而没办法运行呢？答案是肯定的。

思路是从运行php的php-fpm着手，在nginx上做配置，客官请看配置。

<!--more-->

~~~bash
server {
    listen       80;
    server_name  www.example.com;
    root         /home/htdocs/app/;
    index        index.php;
    location / {
        try_files $uri $uri/ /index.php?s=$uri;
    }
    location ~ .*\.php$ {
        # 白名单配置，只允许/index.php、/api.php访问
        if ($fastcgi_script_name !~* "^/(index|api)\.php$") {
            return 403;
        }
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        include        fcgi.conf;
    }
 }
~~~
