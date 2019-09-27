---
title: 设置vim作为crontab -e的默认编辑器
date: 2011-3-15 13:52:47
tags: [vim,crontab, linux]
categories: 后端
---

习惯了用vim修改crontab内容，可有些系统默认的是nano，用起来甚是痛苦，Google搜索了一下，找到下面方法：

<!--more-->

~~~bash
echo "export EDITOR=/usr/bin/vim" >> .bashrc
~~~

然后重新打开终端，输入 `crontab -e`，发现已经默认使用 `vim` 了。
