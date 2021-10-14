---
title: 将 Apple Watch 身份验证添加到 sudo
date: 2021-10-14 13:33:41
tags: [MacOS, Apple Watch, Apple, 身份验证]
categories: MacOS
---

我最近看到一篇关于如何使 `sudo` 与 `Touch ID` 一起工作的文章，这很好，但我的 iMac Pro 没有 `Touch ID`。我继续搜索并找到了`pam-watchid`！

这是一个用于使用 `Watch` 的 `PAM模块` ——正是我想要的。

<!--more-->

它是开源的，因此您可以按照自述文件自行编译，因此请确保您已安装`Xcode`或`Xcode 命令行工具`：

下载[最新的 ZIP 文件](https://github.com/biscuitehh/pam-watchid/archive/main.zip)

解压缩，默认情况下会创建一个名为`pam-watchid-main`的文件夹

- 打开终端并安装它：

```bash
$ cd ~/Downloads/pam-watchid-main
$ sudo 安装
```

- 为sudo注册新的 PAM 模块：

编辑 `/etc/pam.d/sudo`， 在第 1 行（这是注释）下添加一个新行，其中包含：

```text
auth       sufficient     pam_watchid.so
```

（保留此文件中的所有其他行。）
就是这样。现在，无论何时使用sudo，您都可以选择使用 Watch 进行身份验证。

![将 Apple Watch 身份验证添加到 sudo](https://akrabat.com/wp-content/uploads/2020/11/Screenshot-2020-11-22-at-14.17.47-3.png)
