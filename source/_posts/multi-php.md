---
title: 多 PHP 开发环境
date: 2024-07-11 09:12:40
tags: [php, homebrew]
categories: 后端
---

> 本文适用于 M1 或更新版本 macOS 系统，Linux 可参考思路。

## 一、安装 `homebrew`

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 二、安装 PHPs

- 添加仓库源

```shell
brew tap shivammathur/php
```

- 安装指定版本 PHP，以 `7.4` 为例

```shell
brew install shivammathur/php/php@7.4
```

<!--more-->

## 三、快速切换神器

- 脚本位置 `~/scripts/brew-php-switcher.sh`，`~/scripts`可按个人喜好调整

```shell
#!/bin/sh

if [ -z "$1" ]; then
    echo "Usage: brew-php-switcher <version>"
    return 1
fi

version=$1

# 检测参数是否匹配 \d\.\d 格式
if [[ ${version} =~ ^[0-9]\.[0-9]$ ]]; then
    echo "";
else
    echo "参数<version>必须符合 \d\.\d 格式"
    return 1
fi

formula="php@$version"

# 检测参数是否在给定的值列表中
case ${version} in
    7.4|8.0|8.1|8.2|8.4)
        ;;
    8.3) # 默认版本
        formula="php"
        ;;
    *)
        echo "参数<version>不在给定的值列表（7.4|8.0|8.1|8.2|8.3|8.4）中"
        return 1
        ;;
esac

brew list --formula|grep -E "php(\d{2}|@\d\.\d)?$"|xargs brew unlink;
brew link --overwrite --force ${formula} && php -v
```

- `~/.bashrc` 或 `~/.zshrc` 添加以下脚本

```shell
alias brew-php-switcher="sh ~/scripts/brew-php-switcher.sh"
```

- 快速切换

```shell
brew-php-switcher 7.4
```

## 四、自动切换 PHP 版本

- 同样在 `~/.bashrc` 或 `~/.zshrc` 添加以下脚本

```shell
# auto dectect
PROJECT_BASHRC=$(pwd)/.bashrc

# echo $PROJECT_BASHRC
if [ -f $PROJECT_BASHRC ]; then
    source $PROJECT_BASHRC
fi
```

- .bashrc范本，以 PHP `7.4` 项目为例

```shell
export PATH="/opt/homebrew/opt/php@7.4/bin:$PATH"
export PATH="/opt/homebrew/opt/php@7.4/sbin:$PATH"

alias artisan="php artisan"
alias tinker="artisan tinker"
```

> 重启 IDE 即可，记得在项目中忽略 `.bashrc` 文件

## 五、Swoole 安装神器

- 脚本位置 `~/scripts/swoole.sh`

```shell
#!/bin/sh

# 获取包名
function _get_package() {
    local package="swoole"

    if [ "$1" != "" ]; then
        package="swoole-$1"
    fi

    echo $package
}

# 修复 pcre2 路径问题
function _fix_pcre() {
    local php_home=$(php -i | grep -Eo "\-\-prefix=([^']*)" | awk -F '=' '{print $2}')
    local pcre_home=$(brew --prefix pcre2)

    local source="${pcre_home}/include/pcre2.h"
    local target="${php_home}/include/php/ext/pcre/pcre2.h"

    if [ ! -f $target ]; then
        ln -s $source $target
    fi
}

# 安装
function swoole_install() {

    _fix_pcre

    local package=`_get_package $1`

    case $1 in 
        4.8.11)
            pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes --with-openssl-dir=/opt/homebrew/opt/openssl@1.1" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="no"' ${package}
            ;;
        4.8.*)
            pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes --with-openssl-dir=/opt/homebrew/opt/openssl@1.1" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes"' ${package}
            ;;
        5.0.0)
            pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes --with-openssl-dir=/opt/homebrew/opt/openssl@1.1" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="no"' ${package}
            ;;
        5.0.*)
            pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes --with-openssl-dir=/opt/homebrew/opt/openssl@1.1" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="no" enable-brotli="no"' ${package}
            ;;
        5.1.*)
            pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes --with-openssl-dir=/opt/homebrew/opt/openssl@1.1" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="no" enable-brotli="no" enable-swoole-pgsql="no" with-swoole-odbc="no" with-swoole-oracle="no" enable-swoole-sqlite="no"' ${package}
            ;;
        6.0.* | "")
            pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes --with-openssl-dir=/opt/homebrew/opt/openssl@1.1" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="no" enable-brotli="no" enable-swoole-pgsql="no" with-swoole-odbc="no" with-swoole-oracle="no" enable-swoole-sqlite="no" enable-swoole-thread="no" enable-iouring="no"' ${package}
            ;;
        *)
            echo "Unsupported version: $1"
            exit 1
            ;;
    esac

    swoole_info
}

# 更新
function swoole_upgrade() {
    swoole_uninstall
    swoole_install $1
}

# 重新安装
function swoole_reinstall() {
    swoole_uninstall
    swoole_install $1
}

# 卸载
function swoole_uninstall() {
    local php_ini=$(php -i | grep "/php.ini" | awk -F ' => ' '{print $2}')

    pecl uninstall swoole
    sed -i '' '/^extension="swoole.so"/d' $php_ini
}

# 最新版本
function swoole_latest() {
    local version=`curl -s https://pecl.php.net/package/swoole |grep -E "swoole-.*.tgz" |head -1 |grep -Eo '\d+\.\d+\.\d+' | head -1`
    echo $version
}

# 信息
function swoole_info() {
    php --ri swoole
}

# 版本
function swoole_version() {
    php -r "echo swoole_version();" 
}

# 安装别名
function swoole_i() {
    swoole_install $1
}

# 更新别名
function swoole_u() {
    swoole_upgrade $1
}

# 帮助
function swoole_help() {
    cat << END
Usage:
    $0 [help|i|install|reinstall|uninstall|u|upgrade|version|latest] [version]

Examples
    $0 install
    $0 install 5.1.3
    $0 upgrade
    $0 upgrade 5.1.3
END
}

# 入口
function swoole_() {
    swoole_help
}

# 执行
swoole_$1 $2
```

- `~/.bashrc` 或 `~/.zshrc` 添加以下脚本

```shell
alias swoole-tool="sh ~/scripts/swoole.sh"
```

- 快速安装 `Swoole`

```shell
swoole-tool install 6.0.0
```
