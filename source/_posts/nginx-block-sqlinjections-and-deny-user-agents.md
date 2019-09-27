---
title: nginx服务器防sql注入/溢出攻击/spam及禁User-agents
date: 2013-5-4 13:27:10
tags: [nginx, sql, 注入, spam, user-agent, 溢出攻击]
categories: 后端
---

废话不说，直接上配置，客官请看

<!--more-->
~~~bash
server {
    ## 禁SQL注入 Block SQL injections
    set $block_sql_injections 0;
    if ($query_string ~ "union.*select.*\(") {
        set $block_sql_injections 1;
    }
    if ($query_string ~ "union.*all.*select.*") {
        set $block_sql_injections 1;
    }
    if ($query_string ~ "concat.*\(") {
        set $block_sql_injections 1;
    }
    if ($block_sql_injections = 1) {
        return 444;
    }

    ## 禁掉文件注入
    set $block_file_injections 0;
    if ($query_string ~ "[a-zA-Z0-9_]=http://") {
        set $block_file_injections 1;
    }
    if ($query_string ~ "[a-zA-Z0-9_]=(\.\.//?)+") {
        set $block_file_injections 1;
    }
    if ($query_string ~ "[a-zA-Z0-9_]=/([a-z0-9_.]//?)+") {
        set $block_file_injections 1;
    }
    if ($block_file_injections = 1) {
        return 444;
    }

    ## 禁掉溢出攻击
    set $block_common_exploits 0;
    if ($query_string ~ "(<|%3C).*script.*(>|%3E)") {
        set $block_common_exploits 1;
    }
    if ($query_string ~ "GLOBALS(=|\[|\%[0-9A-Z]{0,2})") {
        set $block_common_exploits 1;
    }
    if ($query_string ~ "_REQUEST(=|\[|\%[0-9A-Z]{0,2})") {
        set $block_common_exploits 1;
    }
    if ($query_string ~ "proc/self/environ") {
        set $block_common_exploits 1;
    }
    if ($query_string ~ "mosConfig_[a-zA-Z_]{1,21}(=|\%3D)") {
        set $block_common_exploits 1;
    }
    if ($query_string ~ "base64_(en|de)code\(.*\)") {
        set $block_common_exploits 1;
    }
    if ($block_common_exploits = 1) {
        return 444;
    }

    ## 禁spam字段
    set $block_spam 0;
    if ($query_string ~ "\b(ultram|unicauca|valium|viagra|vicodin|xanax|ypxaieo)\b") {
        set $block_spam 1;
    }
    if ($query_string ~ "\b(erections|hoodia|huronriveracres|impotence|levitra|libido)\b") {
        set $block_spam 1;
    }
    if ($query_string ~ "\b(ambien|blue\spill|cialis|cocaine|ejaculation|erectile)\b") {
        set $block_spam 1;
    }
    if ($query_string ~ "\b(lipitor|phentermin|pro[sz]ac|sandyauer|tramadol|troyhamby)\b") {
        set $block_spam 1;
    }
    if ($block_spam = 1) {
        return 444;
    }

    ## 禁掉user-agents
    set $block_user_agents 0;

    # Don’t disable wget if you need it to run cron jobs!
    # if ($http_user_agent ~ "Wget") {
    #     set $block_user_agents 1;
    # }

    # Disable Akeeba Remote Control 2.5 and earlier
    if ($http_user_agent ~ "Indy Library") {
        set $block_user_agents 1;
    }

    # Common bandwidth hoggers and hacking tools.
    if ($http_user_agent ~ "libwww-perl") {
        set $block_user_agents 1;
    }
    if ($http_user_agent ~ "GetRight") {
        set $block_user_agents 1;
    }
    if ($http_user_agent ~ "GetWeb!") {
        set $block_user_agents 1;
    }
    if ($http_user_agent ~ "Go!Zilla") {
        set $block_user_agents 1;
    }
    if ($http_user_agent ~ "Download Demon") {
        set $block_user_agents 1;
    }
    if ($http_user_agent ~ "Go-Ahead-Got-It") {
        set $block_user_agents 1;
    }
    if ($http_user_agent ~ "TurnitinBot") {
        set $block_user_agents 1;
    }
    if ($http_user_agent ~ "GrabNet") {
        set $block_user_agents 1;
    }

    if ($block_user_agents = 1) {
        return 444;
    }
}
~~~

说明：
> 防sql注入/溢出攻击/spam采用禁掉URL中包含的字段实现，请根据需要合理添加返回444,是完全不回应客户端，比403更加非常节省系统资源
