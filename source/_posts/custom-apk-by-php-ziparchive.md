---
title: 实现自定义apk安装包
date: 2014-8-22 09:05:02
tags: [php, zip, ziparchive, apk]
categories: 后端
---

需求：
> 突然收到老大的需求，要对产品进行一次推荐好友安装的活动，每个会员下载自己的专属安装包（里面记录会员的相关信息）。

思路：
> 经过了解，发现apk安装包原来只是zip的一个马甲，使用php的ZipArchive类可以对文件进行操作。

<!--more-->

实现代码：

~~~php
// 源文件
$apk    = "gb.apk";
// 生成临时文件
$file   = tempnam("tmp", "zip");
// 复制文件
if(false===file_put_contents($file, file_get_contents($apk))){
    exit('copy faild!');
}
// 打开临时文件
$zip    = new ZipArchive();
$zip->open($file);
// 添加文件
// 由于apk限定只能修改此目录内的文件，否则会报无效apk包
$zip->addFromString('META-INF/extends.json', json_encode(array('author'=>'deeka')));
// 关闭zip
$zip->close();
// 下载文件
header("Content-Type: application/zip");
header("Content-Length: " . filesize($file));
header("Content-Disposition: attachment; filename=\"{$apk}\"");
// 输出二进制流
readfile($file);
// 删除临时文件
unlink($file);
~~~
