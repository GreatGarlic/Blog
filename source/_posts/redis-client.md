---
title: Redis 客户端 Redis-Commander
date: 2016-04-14 08:59:29
tags: Redis
---

Redis 有很多可视化的客户端，如用 Qt 实现的，跨平台的 `Redis Desktop Manager`，这里介绍的是 Node 实现的 `Redis-Commander`。

<!--more-->

其官方网址为 <https://www.npmjs.com/package/redis-commander>

## 安装
> 需要先安装 Node 环境

```
npm install -g redis-commander
```

## 启动
```
redis-commander
```

## 访问
<http://127.0.0.1:8081>

* 左边列出 key
* 最下面可输入 Redis 命令

## 帮助
```
$ redis-commander --help
Options:
  --redis-port                    The port to find redis on.         [string]
  --redis-host                    The host to find redis on.         [string]
  --redis-socket                  The unix-socket to find redis on.  [string]
  --redis-password                The redis password.                [string]
  --redis-db                      The redis database.                [string]
  --http-auth-username, --http-u  The http authorisation username.   [string]
  --http-auth-password, --http-p  The http authorisation password.   [string]
  --port, -p                      The port to run the server on.     [string]  [default: 8081]
  --address, -a                   The address to run the server on   [string]  [default: 0.0.0.0]
```
