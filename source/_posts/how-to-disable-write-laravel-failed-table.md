---
title: 如何取消 failed_jobs
date: 2019-11-29T00:24:06.091Z
tags: [Laravel, queue, faild_jobs]
categories: 后端
---

## 烦恼从何而来

用 Laravel 的小伙伴应该都会用到 Queue， 从[手册](https://laravel.com/docs/6.x/queues#dealing-with-failed-jobs) Job 失败后会将队列信息记录到 `failed_jobs` 表，可以通过

~~~bash
php artisan queue:failed-table
php artisan migrate
~~~

作用主要是为了方便分析失败原因和 Job 重试（本身支持 retry），这都很好理解。当 `failed_jobs` 没有被创建的时候，会报这样一个异常：

~~~php
SQLSTATE[42S02]: Base table or view not found: 1146 Table 'a8591.failed_jobs' doesn't exist
~~~

并且记录到 log 文件，如果你的项目中有异常通知，那是相当困扰。

但是有一个场景，也许我并不关心任务执行是否成功，或者说因为某种特定不可控因素允许任务存在执行失败的情况，而我又不希望被这类异常打扰要怎么办呢？

<!--more-->

## 方法一：重写 Job 类 fail 方法

只要 IDE 强大些，跳跳跳，跳转一下代码，很容易找到 `vendor/illuminate/queue/Jobs/Job.php` 处理 Job 失败的逻辑位置：

~~~php
// ...
public function fail($e = null)
{
    $this->markAsFailed();

    if ($this->isDeleted()) {
        return;
    }

    try {
        // If the job has failed, we will delete it, call the "failed" method and then call
        // an event indicating the job has failed so it can be logged if needed. This is
        // to allow every developer to better keep monitor of their failed queue jobs.
        $this->delete();

        $this->failed($e);
    } finally { // 没错，就是这里处理失败任务
        $this->resolve(Dispatcher::class)->dispatch(new JobFailed(
            $this->connectionName, $this, $e ?: new ManuallyFailedException
        ));
    }
}
// ...
~~~

最粗暴的方法就是在具体的 XXXJob 里重写这个方法。

这个方法有一个好处：可以针对某个 Job，而不是一刀切。

## 方法二：修改 queue 配置

后来我再想，要么每个 Job 都要重写一遍，要么再项目中定义一个 Job 父类，然后再父类中重写，方法可行，但是感觉不太优雅，比较 Laravel 是一个这么优雅的框架，怎么可以就这么粗暴处理呢。

于是在 `方法一` 的基础上再深入研究，想要看看有没有`开关`，但是从 queue 的配置文件和手册中并没有明确的提到这个`开关`，只能继续`跳跳跳`，继续深入的了解 Job 的执行原理。

功夫不负有心人，终于让我找到了 `vendor/illuminate/queue/QueueServiceProvider.php`

~~~php
protected function registerFailedJobServices()
{
    $this->app->singleton('queue.failer', function () {
        $config = $this->app['config']['queue.failed'];

        if (isset($config['driver']) && $config['driver'] === 'dynamodb') {
            return $this->dynamoFailedJobProvider($config);
        } elseif (isset($config['table'])) {
            return $this->databaseFailedJobProvider($config);
        } else {
            return new NullFailedJobProvider;
        }
    });
}
~~~

问题变得清晰明朗了，不难看出：

* 当 `queue.failed.driver` 为 `dynamodb` 的时候使用 `dynamoFailedJobProvider` 处理
* 当存在 `queue.failed.table` 的时候，使用 `databaseFailedJobProvider` 处理
* 否则使用 `NullFailedJobProvider` 处理，也就是啥都不干的意思

最终发现原来`开关`是存在的，只是官方没有明确说明。

找到 `config/queue.php`, do it!

~~~php
// ...
/*
|--------------------------------------------------------------------------
| Failed Queue Jobs
|--------------------------------------------------------------------------
|
| These options configure the behavior of failed queue job logging so you
| can control which database and table are used to store the jobs that
| have failed. You may change them to any database / table you wish.
|
*/

'failed' => [
    // 'database' => env('DB_CONNECTION', 'mysql'),
    // 'table' => env('QUEUE_FAILED_TABLE', 'failed_jobs'),
],
~~~
