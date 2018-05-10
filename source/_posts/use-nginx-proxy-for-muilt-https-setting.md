---
title: 利用反向代理批量实现https协议访问
date: 2013-5-14 13:19:10
tags: [nginx, https, proxy]
categories: 后端
---

最近有一个项目需求，需要为站点增加https访问。

开始只配置了www域名下的https，发现css和js都无法正常加载，原因是https页面，如果加载http协议的内容，会被认为页面不安全，尤其是IE，刷新一下页面就要弹出一次确认，相当烦人。

后来苦逼的把各个子域名都加入了https配置，nginx.conf里写各个子域名都写了一个443的server配置，每新增一个域名，还得copy一份，如果是修改一下站点的配置，还得改两次以上。
<!--more-->
后来跟同事讨论，发现使用反向代理，可以快捷现实，配置如下：

~~~bash
sever{
    listen 80;
    sever_name www.hdj.me;
    root /home/webroot/www/;
    index index.php index.html;
    # ...
}
sever{
    listen 80;
    sever_name img.hdj.me;
    root /home/webroot/img/;
    index index.php index.html;
    # ...
}
sever{
    listen 80;
    sever_name static.hdj.me;
    root /home/webroot/static/;
    index index.php index.html;
    # ...
}
sever{
    listen 80;
    sever_name upload.hdj.me;
    root /home/webroot/upload/;
    index index.php index.html;
    # ...
}
server {
    listen 443;
    server_name www.hdj.me img.hdj.me static.hdj.me upload.hdj.me;
    ssl on;
    ssl_certificate /usr/local/nginx/conf/hdj.me.crt;
    ssl_certificate_key /usr/local/nginx/conf/hdj.me.key;
 
    location /{
        proxy_pass http://127.0.0.1:80;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_set_header SSL '1';
        proxy_redirect http:// https://;
    }
}
~~~

这样配置好，访问：
- https://www.hdj.me/
- https://img.hdj.me/
- https://static.hdj.me/
- https://upload.hdj.me/

会被反向代理至：
- http://www.hdj.me/
- http://img.hdj.me/
- http://static.hdj.me/
- http://upload.hdj.me/

而不需要对各个域名都配置一个443的站点。