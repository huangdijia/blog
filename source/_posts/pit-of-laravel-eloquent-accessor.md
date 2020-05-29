---
title: Laravel Eloquent 访问器的一个坑
date: 2020-05-29 09:09:10
tags: [Laravel, Eloquent]
categories: 后端
---

## 数据

|ID|first_name|last_name|
|--|--|--|
|1|Jackie|Chan|
|2|Bruce|Lee|

<!--more-->

## 模型

~~~php

class Star extends Model
{
    public function getFullNameAttribute()
    {
        return $this->first_name . ' ' . $this->last_name;
    }
}

~~~

## 正常使用

~~~php
Start::first()->full_name; // "Jackie Chan"
~~~

## 坑

~~~php
Star::select(DB::raw("concat(first_name, ' ', last_name) as full_name"))->first()->full_name; // ""
~~~

但是

~~~php
dump(Star::select(DB::raw("concat(first_name, ' ', last_name) as full_name"))->first());
~~~

结果能看到 `full_name` 有值

~~~php
// ...
  #attributes: array:1 [
    "full_name" => "Jackie Chan"
  ]
  #original: array:1 [
    "full_name" => "Jackie Chan"
  ]
// ...
~~~

## 原因

> 1. Eloquent 访问器优先级高于属性
> 2. 该例子中访问器依赖的属性因为未指定 `select` 为空

## 如何填坑

~~~php
class Star extends Model
{
    public function getFullNameAttribute()
    {
        // 优先 full_name 属性
        return $this->attributes['full_name'] ?? ($this->attributes['first_name'] . ' ' . $this->attributes['last_name']);
    }
}
~~~
