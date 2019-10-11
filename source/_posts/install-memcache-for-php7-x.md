---
title: 为 PHP7.x 安装 memcache 扩展
date: 2019-10-11 16:46:58
tags: [PHP, Memcache]
categories: 后端
---

[PHP Extension - Memcache module with support of newer PHP 7.0-7.3](https://github.com/websupport-sk/pecl-memcache)

Install

~~~bash
# clone source
git clone https://github.com/websupport-sk/pecl-memcache.git

# configure & install
cd pecl-memcache;
phpize
./configure
make -j$(nproc)
make install

# create php ini
echo "extension=memcache.so" >> /usr/local/php/etc/conf.d/memcache.ini
~~~
