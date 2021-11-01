---
title: MacOS 上安装多版本 PHP
date: 2019-04-10 11:48:47
tags: [macos, php, brew]
categories: 后端
---

## 安装 brew 源

~~~bash
brew tap shivammathur/php
brew search php |grep -E "php@\d\.\d$"
~~~

可以看到结果：

~~~bash
shivammathur/php/php@5.6
shivammathur/php/php@7.0
shivammathur/php/php@7.1
shivammathur/php/php@7.2
shivammathur/php/php@7.3
shivammathur/php/php@7.4
shivammathur/php/php@8.1
shivammathur/php/php@8.2
~~~

<!--more-->

## 安装多版本

跟进需要安装

~~~bash
brew install shivammathur/php/php@5.6
brew install shivammathur/php/php@7.0
brew install shivammathur/php/php@7.1
brew install shivammathur/php/php@7.2
brew install shivammathur/php/php@7.3
brew install shivammathur/php/php@7.4
brew install shivammathur/php/php@8.1
brew install shivammathur/php/php@8.2
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
alias php53='/usr/local/opt/php@5.3/bin/php'
alias php54='/usr/local/opt/php@5.4/bin/php'
alias php55='/usr/local/opt/php@5.5/bin/php'
alias php56='/usr/local/opt/php@5.6/bin/php'
alias php70='/usr/local/opt/php@7.0/bin/php'
alias php71='/usr/local/opt/php@7.1/bin/php'
alias php72='/usr/local/opt/php@7.2/bin/php'
alias php73='/usr/local/opt/php@7.3/bin/php'
alias php74='/usr/local/opt/php@7.4/bin/php'
alias php80='/usr/local/opt/php/bin/php'
# pecl alias
alias pecl53='/usr/local/php5/bin/pecl'
alias pecl71='/usr/local/opt/php@7.1/bin/pecl'
alias pecl72='/usr/local/opt/php@7.2/bin/pecl'
alias pecl73='/usr/local/opt/php@7.3/bin/pecl'
alias pecl74='/usr/local/opt/php@7.4/bin/pecl'
alias pecl80='/usr/local/opt/php/bin/pecl'
# composer alias
alias composer53='php53 /usr/local/bin/composer'
alias composer70='php70 /usr/local/bin/composer'
alias composer71='php71 /usr/local/bin/composer'
alias composer72='php72 /usr/local/bin/composer'
alias composer73='php73 /usr/local/bin/composer'
alias composer74='php74 /usr/local/bin/composer'
alias composer80='php80 /usr/local/bin/composer'
# 切换版本
alias use-php53='unlink-php; brew link --overwrite --force php@5.3 && php -v'
alias use-php54='unlink-php; brew link --overwrite --force php@5.4 && php -v'
alias use-php55='unlink-php; brew link --overwrite --force php@5.5 && php -v'
alias use-php56='unlink-php; brew link --overwrite --force php@5.6 && php -v'
alias use-php70='unlink-php; brew link --overwrite --force php@7.0 && php -v'
alias use-php71='unlink-php; brew link --overwrite --force php@7.1 && php -v'
alias use-php72='unlink-php; brew link --overwrite --force php@7.2 && php -v'
alias use-php73='unlink-php; brew link --overwrite --force php@7.3 && php -v'
alias use-php74='unlink-php; brew link --overwrite --force php@7.4 && php -v'
alias use-php80='unlink-php; brew link --overwrite --force php && php -v'
~~~

后面可以通过 `use-php[version]` 随意切换版本，或者直接用 `php[version]` 执行。
