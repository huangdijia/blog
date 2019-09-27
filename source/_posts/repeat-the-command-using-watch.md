---
title: 使用 watch 帮你重复执行命令
date: 2018-05-15 07:55:04
tags: [linux, watch]
categories: 后端
---

有时候你需要不断的执行某个命令，追踪其输出产生的变化情况。你可能会写一个死循环来做这件事情：

<!--more-->

~~~bash
while :
do
    clear
    commands
    sleep 1
done
~~~

然而实际上`linux中`有一个 `watch` 命令能够帮你做这件事情。它会定期执行指定的程序并将结果全屏输出。

watch 的使用方法很简单，只需要

~~~bash
watch 命令
~~~

就行了，这样 `watch` 命令会每隔两秒执行一次该该命令，并全屏输出执行结果。

从上图可以看出，第一行中的 `Every 2.0s`: 表示 `watch` 每隔2秒执行一次命令。后面的 `date` 为要执行的命令。再后面的 `T520: Thu May 10 16:55:23 2018` 是主机名以及执行命令的时间。

在下面，从第二行开始就是命令执行的时间了。

通过 `-n INTERVAL` 你也可以设置重复执行命令的间隔时间，比如我可以调整为每5秒中执行一次 `date` 命令

~~~bash
watch -n 5 date
~~~

不仅如此,通过 `-d` 选项, `watch` 还能高亮显示两次输出中不同的部分，这个功能相当实用

~~~bash
watch -d -n 1 date
~~~

除了高亮显示输出中改变的部分外，你也可以设置让 `watch` 发现结果有改变时退出循环执行，方法是使用 `-g/--chgexit` 选项

~~~bash
watch -g free
~~~

默认情况下， `watch` 并不会关心命令的执行结果是否成功

但你可以让 watch 检测命令的返回值，当命令运行返回非0时发出蜂鸣(`-b/–beep`)或者直接退出(`-e/–errexit`)。

~~~bash
watch -e wrong_commands
~~~

最后，若你希望 `watch` 只显示出命令的执行结果，而不要显示第一行的那些信息，那么可以使用 `-t` 选项关闭`title`的显示

~~~bash
watch -t date
~~~
