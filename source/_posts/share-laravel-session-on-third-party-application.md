---
title: 第三方应用共享Laravel项目Session
date: 2018-07-17 12:59:00
tags: [laravel, session]
categories: 后端
---

Laravel 框架越来越被PHP开发者青睐，被应用得越来越广泛，大家都恨不得全部用它来重构项目，boss们当然不会同意，但是我们作为工程师也是不会放弃的，那要怎么办呢？

没错，按模块拆分重构，比如`注册`、`登入`等小模块先重构。

<!--more-->

![撸起袖子加油干](http://www.315che.net/uploads/allimg/170329/225TQ495-0.jpg)

一鼓作气，写完了发现`Session`不兼容，去Google百度了一下，也没找到什么好方案，没办法自己分析一下。

## 先看一下Laravel写的Session是什么

~~~php
s:207:"a:4:{s:3:"abc";i:1531800464;s:6:"_token";s:40:"SPhvkWipry9tCxLrU431ueWE8iHBaMkOMU0acQV6";s:9:"_previous";a:1:{s:3:"url";s:26:"http://yourdomain.com/";}s:6:"_flash";a:2:{s:3:"old";a:0:{}s:3:"new";a:0:{}}}";
~~~

是不是很眼熟？没错，是PHP的序列化，要注意的是2次序列化。我们来验证一下：

~~~php
$sessData = <<<DATA
s:207:"a:4:{s:3:"abc";i:1531800464;s:6:"_token";s:40:"SPhvkWipry9tCxLrU431ueWE8iHBaMkOMU0acQV6";s:9:"_previous";a:1:{s:3:"url";s:26:"http://yourdomain.com/";}s:6:"_flash";a:2:{s:3:"old";a:0:{}s:3:"new";a:0:{}}}";
DATA;

$sessData = unserialize(unserialize($sessData));
var_dump($sessData);
/**
array (
  'abc' => 1531800527,
  '_token' => 'SPhvkWipry9tCxLrU431ueWE8iHBaMkOMU0acQV6',
  '_previous' =>
  array (
    'url' => 'http://yourdomain.com/',
  ),
  '_flash' =>
  array (
    'old' =>
    array (
    ),
    'new' =>
    array (
    ),
  ),
)
 */
~~~

## 设置Laravel Session配置，为共享准备

~~~php
// config/session.php
'encrypt' => false,
'cookie'  => 'PHPSESSID',
'domain'  => 'yourdomain.com',
~~~

## 明文 Cookie

很多时候 `Cookie` 是需要被前端童鞋使用的，但是默认情况下 `Laravel` 在响应头中添加的 `Cookie` 信息是加密过的，类似 `eyJpdiI6IjRwOFMyTkl2aGs2TGt4OUcxYXRNXC9BPT0iLCJ2YWx1ZSI6IkpHN0Fqb0ZSaDFxVHE0OHdFRXdXMHc9PSIsIm1hYyI6Ijc2MTljZDVmZDI1Mjg5MTk3NTBlZGM0MzUxMjUyZjQ5MzcxOGE1MWU4Y2ViZTBlYTY5YWRjZjNkZjUwNzNkMDEifQ%3D%3D`，这种时候就得将那些需要 明文 传输的 `Cookie` 加入到 白名单 中去：

在 `/app/Http/Middleware/EncryptCookies.php` 中的 `$except` 数组中将其加入

~~~php
protected $except = [
    'PHPSESSID',
];
~~~

## 第三方应用兼容 Laravel

思路如下：

读取

> Redis取出（Laravel序列化数据，字符串）->`unserialize` * 2（得到数组）->`SessionSerializer::encode()`（得到序列化数据，字符串）->交给PHP内核处理

写入

> PHP系列化数据->`SessionSerializer::decode()`（得到数组）->`serialize()` * 2->写入Redis

![图片](https://i.imgflip.com/1issnv.jpg)

~~~php
// PHP SESSION 序列化器
class SessionSerializer
{
    public static function encode($array, $safe = true, $method = '')
    {
        $method = empty($method) ? ini_get("session.serialize_handler") : $method;
        switch ($method) {
            case "php":
                return self::serializePhp($array, $safe);
                break;
            case "php_binary":
                return self::serializePhpbinary($array, $safe);
                break;
            default:
                throw new Exception("Unsupported session.serialize_handler: " . $method . ". Supported: php, php_binary");
        }
    }

    public static function serializePhp($array, $safe = true)
    {
        if ($safe) {
            $array = unserialize(serialize($array));
        }
        $raw  = '';
        $line = 0;
        $keys = array_keys($array);
        foreach ($keys as $key) {
            $value = $array[$key];
            $line++;
            $raw .= $key . '|';
            if (is_array($value) && isset($value['huge_recursion_blocker_we_hope'])) {
                $raw .= 'R:' . $value['huge_recursion_blocker_we_hope'] . ';';
            } else {
                $raw .= serialize($value);
            }
            $array[$key] = ['huge_recursion_blocker_we_hope' => $line];
        }
        return $raw;
    }

    private static function serializePhpbinary($array, $safe = true)
    {
        return '';
    }

    public static function decode($session_data, $method = '')
    {
        $method = empty($method) ? ini_get("session.serialize_handler") : $method;
        switch ($method) {
            case "php":
                return self::unserializePhp($session_data);
                break;
            case "php_binary":
                return self::unserializePhpbinary($session_data);
                break;
            default:
                throw new Exception("Unsupported session.serialize_handler: " . $method . ". Supported: php, php_binary");
        }
    }

    private static function unserializePhp($session_data)
    {
        $return_data = [];
        $offset      = 0;
        while ($offset < strlen($session_data)) {
            if (!strstr(substr($session_data, $offset), "|")) {
                throw new Exception("Invalid data, remaining: " . substr($session_data, $offset));
            }
            $pos     = strpos($session_data, "|", $offset);
            $num     = $pos - $offset;
            $varname = substr($session_data, $offset, $num);
            $offset += $num + 1;
            $data                  = unserialize(substr($session_data, $offset));
            $return_data[$varname] = $data;
            $offset += strlen(serialize($data));
        }
        return $return_data;
    }

    private static function unserializePhpbinary($session_data)
    {
        $return_data = [];
        $offset      = 0;
        while ($offset < strlen($session_data)) {
            $num = ord($session_data[$offset]);
            $offset += 1;
            $varname = substr($session_data, $offset, $num);
            $offset += $num;
            $data   = unserialize(substr($session_data, $offset));
            $return_data[$varname] = $data;
            $offset += strlen(serialize($data));
        }
        return $return_data;
    }
}

// Redis Session 驱动
class SessionRedis
{
    protected $lifeTime     = 900;
    // Laravel项目中Session存储要求
    protected $sessionName  = 'your_laravel_app_name:';
    // 共享cookie需要
    protected $cookieDomain = 'yourdomain.com';
    protected $handle       = null;

    public function open($savePath, $sessName)
    {
        $this->handle = new Redis;
        $this->handle->connect('127.0.0.1', 6379);
        return true;
    }

    public function close()
    {
        $this->gc(ini_get('session.gc_maxlifetime'));
        $this->handle->close();
        $this->handle = null;
        return true;
    }

    public function read($sessID)
    {
        if (empty($sessID)) {
            return;
        }
        // 从Redis读取
        $sessData = $this->handle->get($this->sessionName . $sessID);
        // 按Laravel格式反序列化
        $sessData = unserialize(unserialize($sessData));
        // 处理非法格式
        $sessData = is_array($sessData) ? $sessData : [];
        // 按PHP内核需要格式序列化
        $sessData = SessionSerializer::encode($sessData);
        // 注意这里需要返回的是序列化后的字符串
        return $sessData;
    }

    public function write($sessID, $sessData)
    {
        if (empty($sessID)) {
            return;
        }
        // 反序列化为数组
        $sessData = SessionSerializer::decode($sessData);
        // 按Laravel需要的格式序列化
        $sessData = serialize(serialize($sessData));
        // 存入Redis
        return $this->handle->setex($this->sessionName . $sessID, $this->lifeTime, $sessData);
    }

    public function destroy($sessID)
    {
        if (empty($sessID)) {
            return;
        }

        return $this->handle->delete($this->sessionName . $sessID);
    }

    public function gc($sessMaxLifeTime)
    {
        return true;
    }

    public function start()
    {
        ini_set('session.cookie_domain', isset($m[0]) ? $m[0] : $this->cookieDomain);
        ini_set('session.auto_start', 0);
        ini_set('session.use_trans_sid', 0);
        ini_set('session.gc_probability', 1);
        ini_set('session.gc_divisor', 1000);
        ini_set('session.gc_maxlifetime', $this->lifeTime);
        ini_set('session.use_cookies', 1);
        ini_set('session.cookie_path', '/');
        session_module_name('user');
        session_set_save_handler(
            array(&$this, 'open'),
            array(&$this, 'close'),
            array(&$this, 'read'),
            array(&$this, 'write'),
            array(&$this, 'destroy'),
            array(&$this, 'gc')
        );
        session_start();
    }
}

(new SessionRedis())->start();
~~~

## Laravel 兼容第三方应用

以 `Memcache` 为例

> 服务器记得编译 `memcache` 扩展

### 增加 session 驱动

~~~php
<?php

namespace App\Extensions;

use Exception;
use Memcache;

class MemcacheSessionHandler implements \SessionHandlerInterface
{
    protected $handle      = null;
    protected $sessionName = '';
    protected $lifeTime;

    public function __construct()
    {
        $this->sessionName = '';
        $this->lifeTime    = config('session.lifetime');
        $this->handle      = new Memcache;

        collect(config('cache.stores.memcached.servers'))->each(function ($server) {
            $this->handle->addServer($server['host'], $server['port'], true);
        });
    }

    public function open($savePath, $sessionName)
    {
        return true;
    }

    public function close()
    {
        $this->handle->close();
        $this->handle = null;

        return true;
    }

    public function read($sessionId)
    {
        if (empty($sessionId)) {
            return;
        }

        $data = $this->handle->get($this->sessionName . $sessionId);
        // info($data, ['format' => 'php']);

        // php -> laravel
        $data = self::decode($data);
        $data = serialize($data);
        // info($data, ['format' => 'laravel']);

        return $data;
    }

    public function write($sessionId, $data)
    {
        if (empty($sessionId)) {
            return;
        }

        // info($data, ['format' => 'laravel']);

        // laravel->php
        $data = unserialize($data);
        $data = self::encode($data);
        // info($data, ['format' => 'php']);

        return $this->handle->set($this->sessionName . $sessionId, $data, 0, $this->lifeTime);
    }

    public function destroy($sessionId)
    {
        if (empty($sessionId)) {
            return;
        }

        return $this->handle->delete($this->sessionName . $sessionId);
    }

    public function gc($lifetime)
    {
        return true;
    }

    public static function encode($array, $safe = true, $method = '')
    {
        $method = empty($method) ? ini_get("session.serialize_handler") : $method;

        switch ($method) {
            case "php":
                return self::serializePhp($array, $safe);
                break;
            case "php_binary":
                return self::serializePhpbinary($array, $safe);
                break;
            default:
                throw new Exception("Unsupported session.serialize_handler: " . $method . ". Supported: php, php_binary");
        }
    }

    public static function serializePhp($array, $safe = true)
    {
        if ($safe) {
            $array = unserialize(serialize($array));
        }

        $raw  = '';
        $line = 0;
        $keys = array_keys($array);

        foreach ($keys as $key) {
            $value = $array[$key];
            $line++;
            $raw .= $key . '|';
            if (is_array($value) && isset($value['huge_recursion_blocker_we_hope'])) {
                $raw .= 'R:' . $value['huge_recursion_blocker_we_hope'] . ';';
            } else {
                $raw .= serialize($value);
            }
            $array[$key] = ['huge_recursion_blocker_we_hope' => $line];
        }

        return $raw;
    }

    private static function serializePhpbinary($array, $safe = true)
    {
        return '';
    }

    public static function decode($session_data, $method = '')
    {
        $method = empty($method) ? ini_get("session.serialize_handler") : $method;

        switch ($method) {
            case "php":
                return self::unserializePhp($session_data);
                break;
            case "php_binary":
                return self::unserializePhpbinary($session_data);
                break;
            default:
                throw new Exception("Unsupported session.serialize_handler: " . $method . ". Supported: php, php_binary");
        }
    }

    private static function unserializePhp($session_data)
    {
        $return_data = [];
        $offset      = 0;

        while ($offset < strlen($session_data)) {
            if (!strstr(substr($session_data, $offset), "|")) {
                throw new Exception("Invalid data, remaining: " . substr($session_data, $offset));
            }

            $pos     = strpos($session_data, "|", $offset);
            $num     = $pos - $offset;
            $varname = substr($session_data, $offset, $num);
            $offset += $num + 1;
            $data                  = unserialize(substr($session_data, $offset));
            $return_data[$varname] = $data;
            $offset += strlen(serialize($data));
        }

        return $return_data;
    }

    private static function unserializePhpbinary($session_data)
    {
        $return_data = [];
        $offset      = 0;

        while ($offset < strlen($session_data)) {
            $num = ord($session_data[$offset]);
            $offset += 1;
            $varname = substr($session_data, $offset, $num);
            $offset += $num;
            $data                  = unserialize(substr($session_data, $offset));
            $return_data[$varname] = $data;
            $offset += strlen(serialize($data));
        }

        return $return_data;
    }
}
~~~

### 扩展驱动

编辑 app\AppServiceProvider.php，在 `boot()` 方法增加以下代码

~~~php
public function boot() {
    Session::extend('memcache', function ($app) {
        return new \App\Extensions\MemcacheSessionHandler;
    });
}
~~~

### 修改驱动

编辑或增加 `.env`

~~~env
SESSION_DRIVER=memcache
SESSION_LIFETIME=120
SESSION_DOMAIN=.a.com
~~~

## 特别注意 `!!!!`

如果第三方应用 PHP 版本低于 `7.0`，需要设置 session_id 长度与 Laravel 项目（session_id 长度为 40）的一致

~~~php
// PHP_VERSION < 7.0
ini_set('session.hash_function', 1);
ini_set('session.hash_bits_per_character', 4);
~~~

Laravel 设置认证信息也需注意，`auth()->login($user)` 会更新 session_id，可以改用 `auth()->setUser($user)`

最后，如果你有更好的方案，请留言给我，一起交流共同进步。
