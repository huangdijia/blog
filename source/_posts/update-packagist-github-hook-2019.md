---
title: 2019如何更新packagist github-hook
date: 2019-01-04 16:38:58
tags:
---

去年（2018）Packagist 就有提醒：

> This package is using the legacy GitHub service and will stop being auto-updated in early 2019. Please set up the new GitHub Hook for Packagist so that it keeps working in the future.

大概意思是旧的 GitHub WebHook 被放弃了，请大家更新最新版，目前网上没有比较详细的说明，自己摸索了一下：

1. 打开 https://packagist.org/profile/edit ，看是否绑定了 GitHub Account ，如果没有直接绑定。如果绑定过，解除重新绑定（因为要重新授权）。
2. 打开 https://github.com/your-github-account/your-project/settings/hooks ，删除旧的 WebHook（Don't ask why，just do it）。
3. 回到 https://packagist.org/profile/ ，点击 `retry hook sync` 重新同步 WebHook。
4. 再次查看 GitHub WebHook 的时候发现已经自动配置好了，packagist 对应的项目页面也没有 `WebHook` 相关提示了。

> 发布新的 package 也只需要点击 `retry hook sync` 就可以自动绑定了，不需要手动添加。