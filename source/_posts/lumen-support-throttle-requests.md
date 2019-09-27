---
title: 让Lumen支持请求控制
date: 2018-06-11 14:47:57
tags: [lumen, php, throttle]
categories: 后端
---

> Lumen 是你构建微服务架构和 `API` 应用的完美解决方案，事实上，她是全宇宙最快的框架之一，甚至要快过以速度著称的 `Silex` 和 `Slim`，现在，为你的 `Laravel` 应用程序编写微服务架构变得再简单不过了。

但是你在使用的过程中，你会发现很多 `Laravel` 中好用的功能都被精简了，比如说请求控制中间件 `Throttle`。这个中间件能简单的实现请求控制。那么接下来跟我一起为 `Lumen` 重新添加这么好用的功能吧。

<!--more-->

## 从 Laravel 移植

[GitHub地址](https://github.com/laravel/framework/blob/5.2/src/Illuminate/Routing/Middleware/ThrottleRequests.php)

保存为 `app/Http/Middleware/ThrottleRequests.php`，别忘了修改 `namespace` 为 `App\Http\Middleware`，完整代码如下：

~~~php
<?php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Cache\RateLimiter;
use Symfony\Component\HttpFoundation\Response;

class ThrottleRequests
{
    /**
     * The rate limiter instance.
     *
     * @var \Illuminate\Cache\RateLimiter
     */
    protected $limiter;

    /**
     * Create a new request throttler.
     *
     * @param  \Illuminate\Cache\RateLimiter  $limiter
     * @return void
     */
    public function __construct(RateLimiter $limiter)
    {
        $this->limiter = $limiter;
    }

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  int  $maxAttempts
     * @param  int  $decayMinutes
     * @return mixed
     */
    public function handle($request, Closure $next, $maxAttempts = 60, $decayMinutes = 1)
    {
        $key = $this->resolveRequestSignature($request);

        if ($this->limiter->tooManyAttempts($key, $maxAttempts, $decayMinutes)) {
            return $this->buildResponse($key, $maxAttempts);
        }

        $this->limiter->hit($key, $decayMinutes);

        $response = $next($request);

        return $this->addHeaders(
            $response, $maxAttempts,
            $this->calculateRemainingAttempts($key, $maxAttempts)
        );
    }

    /**
     * Resolve request signature.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return string
     */
    protected function resolveRequestSignature($request)
    {
        // return $request->fingerprint();
        // Lumen 版本缺少 Request::fingerprint 方法
        return sha1(
            $request->method() .
            '|' . $request->server('SERVER_NAME') .
            '|' . $request->path() .
            '|' . $request->ip()
        );
    }

    /**
     * Create a 'too many attempts' response.
     *
     * @param  string  $key
     * @param  int  $maxAttempts
     * @return \Illuminate\Http\Response
     */
    protected function buildResponse($key, $maxAttempts)
    {
        $response = new Response('Too Many Attempts.', 429);

        $retryAfter = $this->limiter->availableIn($key);

        return $this->addHeaders(
            $response, $maxAttempts,
            $this->calculateRemainingAttempts($key, $maxAttempts, $retryAfter),
            $retryAfter
        );
    }

    /**
     * Add the limit header information to the given response.
     *
     * @param  \Symfony\Component\HttpFoundation\Response  $response
     * @param  int  $maxAttempts
     * @param  int  $remainingAttempts
     * @param  int|null  $retryAfter
     * @return \Illuminate\Http\Response
     */
    protected function addHeaders(Response $response, $maxAttempts, $remainingAttempts, $retryAfter = null)
    {
        $headers = [
            'X-RateLimit-Limit'     => $maxAttempts,
            'X-RateLimit-Remaining' => $remainingAttempts,
        ];

        if (!is_null($retryAfter)) {
            $headers['Retry-After'] = $retryAfter;
        }

        $response->headers->add($headers);

        return $response;
    }

    /**
     * Calculate the number of remaining attempts.
     *
     * @param  string  $key
     * @param  int  $maxAttempts
     * @param  int|null  $retryAfter
     * @return int
     */
    protected function calculateRemainingAttempts($key, $maxAttempts, $retryAfter = null)
    {
        if (!is_null($retryAfter)) {
            return 0;
        }

        return $this->limiter->retriesLeft($key, $maxAttempts);
    }
}
~~~

## 注册中间件

找到 `bootstrap/app.php` 文件，添加：

~~~php
$app->routeMiddleware([
    'throttle' => App\Http\Middleware\ThrottleRequests::class,
]);
~~~

## 更新 composer

~~~bash
composer dump-autoload
~~~

## 测试

找到 `routes/web.php`，修改 or 添加

~~~php
// 默认速率是60请求/分钟
$router->group(['prefix' => 'api', 'middleware'=>'throttle'], function () use ($router) {
    $router->get('/', function () {
        return $router->app->version();
    });
});
// 自定义速率是2请求/分钟
$router->group(['prefix' => 'home', 'middleware'=>'throttle:2,1'], function () use ($router) {
    $router->get('/', function () {
        return $router->app->version();
    });
});
~~~
