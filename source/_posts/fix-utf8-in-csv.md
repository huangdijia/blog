---
title: 解决PHP输出UTF-8编码的csv乱码问题
date: 2017-07-14 17:29:16
tags: [php, utf-8, csv]
categories: 后端
---

众所周知，UTF-8已经一统天下了，但是微软office默认使用的还不是它，经常在使用程序生成CSV的时候总是看到的是乱码，其实office并非顽固分子，在早期版本已经增加了对UTF-8的支持，但是你要告诉他，你的数据是UTF-8。

怎么做呢？

<!--more-->

## PHP处理方式

~~~php
// utf8
$fp = fopen('./test_csv.csv', 'a');
fwrite($fp,chr(0xEF).chr(0xBB).chr(0xBF));//输出BOM头
fputcsv($fp, ['标题']);
fputcsv($fp, ['解决乱码']);
fclose($fp);
~~~

## 命令行处理方式

~~~bash
printf '\xEF\xBB\xBF’ > tmp.csv
cat your.csv >> tmp.csv
mv tmp.csv your.csv
~~~

你 `get√` 到了吗？
