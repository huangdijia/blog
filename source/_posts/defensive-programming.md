---
title: 防御性编程
date: 2014-06-09 17:59:58
tags: [编程, php]
categories: 后端
---

在公司，我们碰到的很大一部分问题都是NullPointerException。我常常就想：这段程序明明在我电脑运行好好的，为什么会出现这种情况呢？
因为，我们永远都无法预测用户使用时会发生的各种情况。所以防御性编程可以让我们减少很大一部分错误。

<!--more-->

## 开始之前先看这段代码

需求： 将字符串转换成一个数字，如"12345"(string)转为12345(int)。
代码：

~~~php
function atoi($a){
    $len    = strlen($a);
    $num    = 0;
    for($i=$len-1; $i>=0; $i--){
        $num    += ($a{$i}-0) * pow(10, $len-$i-1);
    }
    return $num;
}
~~~

问题：

- 如果传进来一个负数结果是什么？比如："-123456"。
- 如果传进来一个null或false会怎么样？（php返回的结果都是0，貌似没错）

恍然大悟！！！太多东西没考虑！！！

## 什么是防御性编程

> 顾名思义，防御性编程（Defensive programming）是一种细致、谨慎的编程方法。
> 防御式编程主要用于可能被滥用，恶作剧或无意地造成灾难性影响的程序上。
> 防御性编程是一种防卫方式，而不是一种补救形式。

## 如何写防御性代码呢

看看这样会不会好一些？

~~~php
function atoi($a){
    if(is_null($a)) throw new Exception('$a is null');
    if(false===$a) throw new Exception('$a is false');
    if(0==strlen($a)) throw new Exception('$a is empty string');
    $len    = strlen($a);
    $num    = 0;
    for($i=$len-1; $i>=0; $i--){
        $num    += ($a{$i}-0) * pow(10, $len-$i-1);
    }
    return $num;
}
~~~

重要的经验就是：
当你写一个方法需要对传入的参数进行处理或者计算的时候，你必须要严格验证传入参数的正确性，如果不符合，就应当给出提示！
防御性编程的最基本规则：
保护程序免遭非法输入数据的破坏。

## 为什么要防御性编程

- 为了让你写的代码正确运行
- 提供参数错误的原因
- 可以快速找到 bug
