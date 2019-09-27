---
title: Swoole实现即时聊天服务
date: 2018-07-18 09:12:14
tags: [swoole, chat]
categories: 后端
---

## 需求背景

就技术面而言，即时通讯对很多人或公司来说已经没有什么门槛，技术方案有如雨后春笋，也各有千秋，也有不少专业提供第三方服务的公司，如`云信`、`融云`等等，几个大厂（`阿里云`、`腾讯云`）也有提供云服务。

<!--more-->

这里要讲一下的是第三方功能非常强大，也兼容各个平台，但是灵活性不够个性需求无法定制。

刚提到技术上已经没有门槛，开源的项目也有不少，基本都是基于`长连接+轮询`或`websocket`，长连接的代表有`iComet`，而`websocket`的代表有`Swoole`。

花了半天写了个demo，先看看需求：

> * 支持点对点聊天
> * 同一账号支持多端，能同时受到信息

TODO

> * 支持消息广播
> * 支持离线消息

## 服务端

~~~php
<?php

class FdMapping
{
    private $redis;
    const MAP_UID_FD_PREFIX = 'chat_map_uid_fd:';
    const MAP_FD_UID_PREFIX = 'chat_map_fd_uid:';

    public function __construct($redis_host = '127.0.0.1', $redis_port = 6379)
    {
        $this->redis = new Redis();
        $this->redis->connect($redis_host, $redis_port);
    }

    // uid与fd对应，支持多端/多页面
    public function uidBindFd($uid, $fd)
    {
        return $this->redis->sadd(
            self::MAP_UID_FD_PREFIX . $uid,
            $fd
        );
    }

    // fd与uid对应
    public function fdBindUid($fd, $uid)
    {
        return $this->redis->setex(
            self::MAP_FD_UID_PREFIX . $fd,
            24 * 3600,
            $uid
        );
    }

    // 获取fd对应的uid
    public function getUidByFd($fd)
    {
        return $this->redis->get(self::MAP_FD_UID_PREFIX . $fd);
    }

    // 获取uid全部fd，确保多端都能收到信息
    public function getFdsByUid($uid)
    {
        return $this->redis->sMembers(self::MAP_UID_FD_PREFIX . $uid);
    }

    // 删除uid的某个fd
    public function delFd($fd, $uid = null)
    {
        if (is_null($uid)) {
            $uid = $this->getUidByFd($fd);
        }
        if (!$uid) {
            return false;
        }
        $this->redis->srem(self::MAP_UID_FD_PREFIX . $uid, $fd);
        $this->redis->del(self::MAP_FD_UID_PREFIX . $fd);
        return true;
    }
}

$FdMapping = new FdMapping('127.0.0.1');
$server    = new swoole_websocket_server("127.0.0.1", 9502);

$server->on('open', function($server, $req) {
    echo "新连接: #{$req->fd}\n";
});

$server->on('message', function($server, $frame) use ($FdMapping) {
    echo "接收到: " . json_encode($frame->data) . "\n";
    // 解析json为array
    $data = json_decode($frame->data, true);
    // 处理不同的消息类型
    switch ($data['type']) {
        case 'sign':
            // 设置uid与fd对应关系
            $FdMapping->uidBindFd($data['uid'], $frame->fd);
            $FdMapping->fdBindUid($frame->fd, $data['uid']);
            // 清理过期/已断开的连接
            $fds = $FdMapping->getFdsByUid($data['uid']);
            foreach ((array)$fds as $fd) {
                !$server->exist($fd) && $FdMapping->delFd($fd, $data['uid']);
            }
            // 返回注册成功信息，token未验证
            $server->push(
                $frame->fd,
                json_encode([
                    'status' => 1,
                    'msg'    => '注册成功',
                    'token'  => md5(time()), // 可用于连接合法性验证
                ])
            );
            break;
        case 'msg':
            $data['from'] = $FdMapping->getUidByFd($frame->fd);
            // 验证是否已经注册
            if (!$data['from']) {
                $server->push(
                    $frame->fd,
                    json_encode([
                        'status' => 0,
                        'msg'    => '发送失败，未登入',
                    ])
                );
                return;
            }
            // 将消息推送到uid各端
            $fds = $FdMapping->getFdsByUid($data['to']);
            foreach ((array) $fds as $fd) {
                $server->exist($fd) && $server->push(
                    $fd,
                    json_encode([
                        'body' => $data['body'],
                        'from' => $data['from'],
                    ])
                );
                echo "#{$fd} 推送成功\n";
            }
            // 告知推送成功
            $server->push($frame->fd, json_encode(['status' => 1, 'msg' => 'sended']));
            break;
        default:
            // 非法请求
            $server->push(
                $frame->fd,
                json_encode([
                    'status'     => 0,
                    'msg'        => '非法请求',
                    'disconnect' => 1, // 告知终端请断开
                ])
            );
            break;
    }
});

$server->on('close', function($server, $fd) use ($FdMapping) {
    echo "连接断开: #{$fd}\n";
    $FdMapping->delFd($fd);
});

$server->start();
~~~

## 客户端

~~~javascript
var ws = new WebSocket("ws://127.0.0.1:9502");

ws.onopen = function (evt) {
    console.log("Connection open ...");
    ws.send('{"type":"sign", "uid":"1001"}');
};

ws.onmessage = function (evt) {
    console.log("Received Message: " + evt.data);
    // ws.close();
};

ws.onclose = function (evt) {
    console.log("Connection closed.");
};
~~~
