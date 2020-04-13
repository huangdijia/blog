---
title: Laravel artisan 命令自动补全
date: 2020-04-13 13:32:16
tags: [laravel, artisan]
---

## 安装 `bash-completion@2`

~~~bash
brew install bash-completion@2
~~~

<!--more-->

## 配置 `~/.zshrc` 或 `~/.bash_profile`

~~~bash
# bash-completion@2
autoload bashcompinit
bashcompinit

# auto-complete for php artisan
ARTISAN_COMMANDS=`php artisan --raw --no-ansi list | sed "s/[[:space:]].*//g"`
_artisan()
{
    COMP_WORDBREAKS=${COMP_WORDBREAKS//:}
    COMPREPLY=(`compgen -W "$ARTISAN_COMMANDS" -- "${COMP_WORDS[COMP_CWORD]}"`)
    return 0
}
complete -F _artisan artisan
alias artisan='php artisan'
~~~

如果在 Windows 的 Git-Bash 上出现类似于 `stdout is not a tty` 的错误，只需要把 `ARTISAN_COMMANDS=` 这一行改成:

~~~bash
ARTISAN_COMMANDS=`php.exe artisan --raw --no-ansi list | sed "s/[[:space:]].*//g"`
~~~

## 效果预览

~~~bash
>> php artisan make:channel
make:channel       make:factory       make:model         make:rule
make:command       make:import        make:notification  make:seeder
make:controller    make:job           make:observer      make:service
make:event         make:listener      make:policy        make:test
make:exception     make:mail          make:provider      make:transformer
make:export        make:middleware    make:request
make:export-model  make:migration     make:resource
~~~
