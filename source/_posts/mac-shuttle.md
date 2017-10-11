---
title: Shuttle
date: 2016-06-29 16:40:50
tags: Mac
---

使用 `Shuttle` 来管理命令的快捷方式，例如 SSH 的登录等。

主页: <http://fitztrev.github.io/shuttle/>

<!--more-->

## 支持的终端程序
* 默认情况下，使用系统自带的终端 `Terminal.app`
* 切换到 `iTerm2`，edit ~/.shuttle.json，并且改变 `terminal` 的属性

## 配置
配置位于 `<User>/.shuttle`  
主要是配置 `hosts`:

* `cmd`: 命令
* `name`: 显示在 Shuttle 中的标题，点击它后执行 cmd 里的命令

```json
{
    "_comments": [
        "Valid terminals include: 'Terminal.app' or 'iTerm'",
        "In the editor value change 'default' to 'nano', 'vi', or another terminal based editor.",
        "Hosts will also be read from your ~/.ssh/config or /etc/ssh_config file, if available",
        "For more information on how to configure, please see http://fitztrev.github.io/shuttle/"
    ],
    "editor": "default",
    "launch_at_login": false,
    "terminal": "Terminal.app",
    "iTerm_version": "nightly",
    "default_theme": "default_theme",
    "open_in": "tab",
    "show_ssh_config_hosts": false,
    "ssh_config_ignore_hosts": [],
    "ssh_config_ignore_keywords": [],

    "hosts": [{
        "name": "Server 1",
        "cmd": "echo '    登陆 XXX'; ssh root@219.146.255.179"
    }, {
        "name": "Server 2",
        "cmd": "echo '    登陆 YYY'; ssh root@219.146.255.179"
    }]
}
```
