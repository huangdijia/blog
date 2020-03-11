---
title: 通过 JavaScript 监听暗黑模式变化
date: 2020-03-11 07:56:41
tags: [javascript, dark-mode]
categories: 前端
---

苹果系统（macOS、iOS）都已经全面支持暗黑模式（也叫深色模式）了，在这里不做过多的介绍了。在众多的技术文章里都有介绍如何使用，最常见的是通过 CSS 中的 prefers-color-scheme: dark 来检测用户是否开启了 Dark Mode，在 CSS里定义不同的样式。

<!--more-->

~~~css
/* 操作系统及浏览器未支持或用户未开启 Dark Mode */
body {
    background-color: white;
    color: black;
}

@media (prefers-color-scheme: dark) {
    /* 操作系统及浏览器支持且用户开启了 Dark Mode */
    body {
        background-color: black;
        color: white;
    }
}
~~~

从以上代码看，有点类似 if-else，兼容性和可读性都不是很好。

暗黑模式刚推出的时候，我首先想到的是跟博客的主题功能很像，细品之后发现区别在于主题做法更多的是是更换 CSS 文件，于是我想是不是可以通过 JavaScript 检测（监听）用户是否启用暗黑模式，从而加载不同的样式，实现实时更换主题功能。

接下来最关键的问题就是怎么实现这个“监听”功能，在谷歌上百度了一下，找到了一个关键词 matchMedia。

用过 matchMedia 方法可以直接判断浏览器当前是否倾向于使用深色模式：

~~~javascript
let prefersDarkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
if (prefersDarkMode) {
    // 搞事情
}
~~~

通过监听 matchMedia 的 change 事件，可以在用户切换深色 / 浅色模式的时候，将浏览器中已打开的页面自动切换为系统对应的模式。

~~~javascript
let media = window.matchMedia('(prefers-color-scheme: dark)');
let callback = (e) => {
    let prefersDarkMode = e.matches;
    if (prefersDarkMode) {
        // 搞事情
    }
};
if (typeof media.addEventListener === 'function') {
    media.addEventListener('change', callback);
} else if (typeof media.addListener === 'function') {
    media.addListener(callback);
}
~~~
