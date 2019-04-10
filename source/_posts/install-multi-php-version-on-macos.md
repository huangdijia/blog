---
title: MacOS 上安装多版本 PHP
date: 2019-04-10 11:48:47
tags: [macos, php, brew]
categories: 后端
---

## 安装 brew 源

~~~bash
brew tap kyslik/homebrew-php
brew search php |grep -E "php\d{2}$"
~~~

可以看到结果：

~~~bash
kyslik/php/php53
kyslik/php/php54
kyslik/php/php55
kyslik/php/php56
kyslik/php/php70
kyslik/php/php71
~~~

<!--more-->

## 安装多版本

跟进需要安装

~~~bash
brew install kyslik/php/php53
brew install kyslik/php/php54
brew install kyslik/php/php55
brew install kyslik/php/php56
brew install kyslik/php/php70
brew install kyslik/php/php71
~~~

## 命令别名

~~~bash
vim ~/.zshrc
~~~

或

~~~bash
vim ~/.bash_profile
~~~

加入以下内容

~~~bash
# 解除所有关联
alias unlink-php='brew list|grep -E "php(\d{2}|@\d\.\d)$"|xargs brew unlink; brew unlink php'
# 命令别名
alias php53='/usr/local/opt/php53/bin/php'
alias php54='/usr/local/opt/php54/bin/php'
alias php55='/usr/local/opt/php55/bin/php'
alias php56='/usr/local/opt/php@5.6/bin/php'
alias php70='/usr/local/opt/php70/bin/php'
alias php71='/usr/local/opt/php@7.1/bin/php'
alias php72='/usr/local/opt/php@7.2/bin/php'
alias php73='/usr/local/opt/php/bin/php'
# 切换版本
alias use-php53='unlink-php; brew link --overwrite --force php53 && php -v'
alias use-php54='unlink-php; brew link --overwrite --force php54 && php -v'
alias use-php55='unlink-php; brew link --overwrite --force php55 && php -v'
alias use-php56='unlink-php; brew link --overwrite --force php@5.6 && php -v'
alias use-php70='unlink-php; brew link --overwrite --force php@7.0 && php -v'
alias use-php71='unlink-php; brew link --overwrite --force php@7.1 && php -v'
alias use-php72='unlink-php; brew link --overwrite --force php@7.2 && php -v'
alias use-php73='unlink-php; brew link --overwrite --force php && php -v'
~~~

后面可以通过 use-php[version] 随意切换版本，或者直接用 php[version] 执行。