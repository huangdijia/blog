---
title: PHP 的 __call 和 __callStatic
date: 2019-10-15 08:28:35
tags: [PHP]
categories: 后端
---

~~~php
class Foo
{
    public function __call($name, $arguments)
    {
        echo 1;
    }

    public static function __callStatic($name, $arguments)
    {
        echo 2;
    }

    public function bar()
    {
        Foo::abc();
    }
}

(new Foo)->bar();
~~~

结果是 `1` 还是 `2`？答案是 `1`

接下来解释一下：

1. `__call` 方法关注方法能否被访问到，而不仅仅是关注是否存在
2. `__callStatic` 方法关注的是方法能否被静态的访问到，而不是关注方法是否存在，是否是静态方法。
3. 具体执行 `__call`，`__callStatic` ，是根据调用的上下文。如
