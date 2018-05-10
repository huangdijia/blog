---
title: 修改配置nginx限制恶意爬虫频率
date: 2013-5-14 13:25:18
tags: [nginx, 爬虫, robots]
categories: 后端
---
超过设置的限定频率，就会给`spider`一个`503`。
上述配置详细解释请自行`google`下，具体的`spider/bot`名称请自定义。

<!--more-->
~~~bash
#全局配置
limit_req_zone $anti_spider zone=anti_spider:10m rate=15r/m;
#某个server中
limit_req zone=anti_spider burst=30 nodelay;
if ($http_user_agent ~* "xxspider|xxbot") {
  set $anti_spider $http_user_agent;
}
~~~
