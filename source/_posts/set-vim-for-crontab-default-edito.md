---
title: 设置vim作为crontab -e的默认编辑器
date: 2011-3-15 13:52:47
tags: [vim,crontab, linux]
categories: 后端
---
~~~bash
echo "export EDITOR=/usr/bin/vim" >> .bashrc
~~~

然后重新打开终端，输入crontab -e，发现已经默认使用vim了。