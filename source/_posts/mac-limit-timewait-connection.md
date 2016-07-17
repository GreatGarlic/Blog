---
title: 限制 TIME_WAIT 的连接数
date: 2016-07-17 22:32:23
tags: Mac
---

`TIME_WAIT` 连接出现在主动关闭连接一方(TCP 四次握手断开连接)，在大并发的时候，如果配置不当，系统就会出现大量 `TIME_WAIT` 的连接，导致系统响应慢，甚至无响应，可以修改系统内核的参数对其进行限制。

<!--more-->

1. `vi /etc/sysctl.conf`
2. `net.ipv4.tcp_max_tw_buckets = 8000` (限制为 8000，默认为 180000)
3. `sysctl -p` 使其生效
4. 大并发测试: 
    1. `ab -n20000 -c200 http://host:8080/` (Linux 安装 ab: `yum install httpd-tools`)
    2. `netstat -n | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,"\t",state[key]}'` (查看每种状态的连接数)
