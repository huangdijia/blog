---
title: 如何正确的使用 GitHub Actions 实现 Hexo 博客的 CICD
date: 2019-09-27 12:42:48
tags: [Github, actions]
categories: 后端
---

> CI/CD (continuous integration and continuous deployment)

最近同事分享了 GitHub Actions ，发现除了我们常听到的自动部署（项目上线）外还可以做很多事情，比如说就可以自动生成 Hexo 博客，我擦！这不就是我想要的吗？以后就不用每次写完文章之后再执行 `hexo clean && hexo g -d` 了。

回来各种百度，有几篇针对 HEXO 的文章，照着开始写脚本，发现怎么都不能成功，各种错误。折腾了 3 个小时，终于顺利通过了。

<!--more-->

## 项目背景

我先介绍一下我的项目背景：

|项目|说明|
|--|--|
|`https://github.com/huangdijia/blog`| 用于存放 hexo 生成的项目，可以理解成源码|
|`https://github.com/huangdijia/huangdijia.github.io`| 存放 hexo 编译后的静态文件，也是我的博客页面|

> 以下说明都是以我的两个项目为例子

## 密钥生成

因为 hexo 编译后需要 push 到 huangdijia.github.io 上，需要 GitHub 的账号密码（尝试过不行）或者密钥，我们需要用 `ssh-keygen` 命令生成一组私钥（没有后缀名）和公钥（.pub结尾）

~~~bash
ssh-keygen -f github-deploy-key # 三次回车即刻
~~~

会生成 `github-deploy-key` 和 `github-deploy-key.pub` 两个文件。

## 配置 GitHub 仓库

### 配置 blog 仓库

打开 `https://github.com/huangdijia/blog/settings/secrets` 点击 `Add new secrets`，分别在

* `Name` 输入 `HEXO_DEPLOY_PRI`
* `Value` 输入前面生成的私有 KEY `github-deploy-key` 的内容

### 配置 huangdijia.github.io 仓库

打开 `https://github.com/huangdijia/huangdijia.github.io/settings/keys`，点击 `Add deploy key`，分别在

* `Title` 输入 `HEXO_DEPLOY_PUB`
* `Key` 输入前面生成的私有 KEY `github-deploy-key.pub` 的内容

## 编写 Action 脚本

使用前先要[申请](https://github.com/features/actions)，然后点击 `Actions` -> `Set up this workflow` 或直接打开 `https://github.com/huangdijia/huangdijia.github.io/actions/new`

~~~yaml
name: HEXO CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [10.x]

    steps:
      - uses: actions/checkout@v1

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}

      - name: Configuration environment
        env:
          HEXO_DEPLOY_PRI: ${{secrets.HEXO_DEPLOY_PRI}}
        run: |
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name "yourname"
          git config --global user.email "your@email.com"

      - name: Install dependencies
        run: |
          npm i -g hexo-cli
          npm i

      - name: Deploy hexo
        run: |
          hexo g -d
~~~

## 修改 _config.yml repo

如果原来是 http 的，要改为 ssh 格式

如：

~~~yaml
deploy:
  # repo: https://github.com/huangdijia/huangdijia.github.io
  repo: git@github.com:huangdijia/huangdijia.github.io.git
~~~

## 享受成果

在 `blog` 项目任意 push，在 `https://github.com/huangdijia/blog/actions` 都有响应的执行记录

## 注意

> 切记不要把账号密码写在脚本上!!!
