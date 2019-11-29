---
title: php74-preload
date: 2019-11-29T05:23:00.654Z
tags: [php, preload]
categories: 后端
---

期待已久的 [PHP7.4](https://www.php.net/archive/2019.php#2019-11-28-1) 终于发布了，个人最期待的功能还是 [Opcache Preloading](https://www.php.net/manual/zh/opcache.configuration.php#ini.opcache.preload)

<!--more-->

包含 2 配置参数：

* opcache.preload string

> 指定要在服务器启动时期进行编译和缓存的 PHP 脚本文件， 这些文件也可能通过 include 或者 opcache_compile_file() 函数 来预加载其他文件。 所有这些文件中包含的实体，包括函数、类等，在服务器启动的时候就被加载和缓存， 对于用户代码来讲是“开箱可用”的。

* opcache.preload_user string

> 考虑到安全因素，禁止以 root 用户预加载代码。该指令方便以其他用户预加载。

怎么尝鲜呢？

## 自己实现

vi vendor/proload.php

~~~php
<?php

$files = require __DIR__ . '/composer/autoload_classmap.php';

foreach (array_unique($files) as $file) {
    opcache_compile_file($file);
}
~~~

## 第三方包

Github 上也有大神放出 composer 扩展包 [Composer-Preload](https://github.com/Ayesh/Composer-Preload)，可通过简单配置，执行 `composer preload` 生成 `vendor/preload.php`

增加 composer.json 配置 `extra`：

~~~json
{
    "extra": {
        "preload": {
            "paths": [
                "web"
            ],
            "exclude": [
                "web/core/tests",
                "web/core/lib/Drupal/Component/Assertion",
                "web/core/modules/simpletest",
                "web/core/modules/editor/src/Tests"
            ],
            "extensions": ["php", "module", "inc", "install"],
            "exclude-regex": "/[A-Za-z0-9_]test\\.php$/i",
            "no-status-check": false,
            "files": [
                "somefile.php"
            ]
        }
    }
}
~~~

## 期待官方支持

已经有人向 composer 提交了[Issue](https://github.com/composer/composer/issues/7777)，值得期待
