---
title: 如何迁移 homebrew 至 M1 Mac
date: 2021-11-25 08:12:42
tags: [macos, brew]
---

## 旧 Mac 备份

```bash
/usr/local/bin/brew bundle dump 
```

得到 `Brewfile` 文件。

<!--more-->

## M1 Mac 安装 brew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

安装过程中可能会遇到权限问题，可以使用 sudo 命令授权

```bash
sudo chown -R $(whoami) /opt/homebrew
```

## 配置环境变量

编辑 `~/.zshrc` 文件，添加如下内容：

```bash
# arm brew environment
eval "$(/opt/homebrew/bin/brew shellenv)"
```

## 导入 Brewfile

```bash
/opt/homebrew/bin/brew bundle --file path/to/Brewfile
```

> 有些软件不能完成安装，后面手动安装就好了。

## 卸载旧版本

如果旧版跟谁 Mac 也迁移至了 M1 Mac，那么可以使用以下命令卸载

```bash
arch -x86_64 $SHELL
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall.sh)"
```

可能还会有一些残留目录，只要手动 `rm -rf` 删除即可

> /usr/local/.com.apple.installer.keep
> /usr/local/Frameworks/
> /usr/local/Homebrew/
> /usr/local/bin/
> /usr/local/etc/
> /usr/local/iODBC.odbcmanager/
> /usr/local/include/
> /usr/local/lib/
> /usr/local/opt/
> /usr/local/packager/
> /usr/local/remotedesktop/
> /usr/local/sbin/
> /usr/local/share/
> /usr/local/var/
> /usr/local/zlib/
