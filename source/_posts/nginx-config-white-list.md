---
title: nginx安全配置之白名单设置
date: 2014-1-6 13:16:46
tags: [nginx, webserver, php]
categories: 后端
---

~~~bash
server {
    listen       5080;
    server_name  www.example.com;
    root         /home/htdocs/app/;
    index        index.php;
    location / {
        try_files $uri $uri/ /index.php?s=$uri;
    }
    location ~ .*\.php$ {
        # 白名单配置，只允许/index.php访问
        if ($fastcgi_script_name !~* "^/(index)\.php$") {
            return 403;
        }
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        include        fcgi.conf;
    }
 }
~~~