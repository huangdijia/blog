---
title: 让Lumen支持Session
date: 2018-06-08 12:39:36
tags: [lumen, session, php]
categories: 后端
---

[官方介绍](https://lumen.laravel-china.org/docs/5.3/authentication)

> Lumen 虽然与 Laravel 使用了相同的底层类库实现，但是因 Lumen 面向的是无状态 API 的开发，不支持 session，所以默认的配置不同。Lumen 必须使用无状态的机制来实现，如 API 令牌（Token）。

也就是说Lumen内核已经剥离了Cookie、Session，如果需要使用到，需要增加安装，经过一阵折腾，终于整好了，顺便记录下来。

<!--more-->

## 增加session配置

~~~php
return [
    'driver'          => 'file',
    'lifetime'        => 120,
    'expire_on_close' => false,
    'encrypt'         => false,
    'files'           => storage_path('framework/sessions'),
    'connection'      => 'session',
    'table'           => 'sessions_laravel',
    'store'           => null,
    'lottery'         => [2, 100],
    'cookie'          => 'PHPSESSID',
    'path'            => '/',
    'domain'          => env('SESSION_DOMAIN', null),
    'secure'          => false,
    'http_only'       => true,
];
~~~

## 注册Session中间件及服务提供者

> 修改bootstrap/app.php，增加以下内容

~~~php
// 注册Session中间件
$app->middleware([
    \Illuminate\Session\Middleware\StartSession::class,
]);
// 注册Session服务提供者
$app->register(Illuminate\Session\SessionServiceProvider::class);
// 加载Session配置
$app->configure('session');
// 注册容器（Lumen取消默认支持）
$app->bind('Illuminate\Session\SessionManager', 'session');
~~~

## 测试

编辑routes/web.php

~~~php
$router->get('/', function () {
    $key = 'key';
    if (app('session')->has($key)) {
        return app('session')->get($key);
    }
    app('session')->put($key, 'value');
    return 'set session success';
});
~~~

> 生命的意义在于折腾！
