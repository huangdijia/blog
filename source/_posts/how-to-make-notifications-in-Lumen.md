---
title: 如何正确的在 Lumen 中启用 Notification
date: 2019-11-15 18:30:27
tags: [lumen, notification]
category: 后端
---

## 安装 `illuminate/notifications`

~~~bash
composer require illuminate/notifications
~~~

## `bootstrap/app.php` 注册服务

~~~php
$app->register(Illuminate\Notifications\NotificationServiceProvider::class);
~~~

> ⚠️ 必须在 AppServiceProvider 注册之前，不然自定义 `Channel` 会无法找到，提升 `InvalidArgumentException with message 'Driver [xxxx] not supported.'`

## 在 `AppServiceProvider.php` 注册 `Channel`

~~~php
$this->app->make(Illuminate\Notifications\ChannelManager::class)->extend('your-channel', function() {
    return $this->app->make(App\Channels\YourChannel::class);
});
~~~
