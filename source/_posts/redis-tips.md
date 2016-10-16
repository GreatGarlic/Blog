---
title: Redis Tips
date: 2016-07-05 21:35:23
tags: [Java, Util, Redis]
---

## 启动 Redis (默认配置)
```
redis-server
```

## 启动 Redis (指定配置)
```
redis-server redis.conf
```

## 启动 Redis (非保护模式)
```
redis-server --protected-mode no
```

<!--more-->

## 启动 Redis (后台运行)
```
1. 修改配置: daemonize yes
2. 启动服务: redis-server redis.conf
```

## 关闭 Redis
```
redis-cli shutdown
```

## 测试链接 Redis
```
telnet redis-server-ip 6379
```

## 设置密码
```
requirepass 123456
```

1. 重启 Redis 服务
1. 打开 Redis 客户端: `redis-cli`
2. `keys *` 提示错误 `(error) NOAUTH Authentication required.` 
3. `auth 123456`，然后 `keys *`，输出所有的 key

## 绑定 IP
Redis 采用的安全策略默认会只准许本机访问，修改 bind 配置可以允许外网访问，`但是 bind 的是 Redis 所在服务器网卡的 ip`，也就是说，如果你的 Redis 服务器有两张网卡，一张是 ip-1,另一张是 ip-2，如果 `bind ip-1`，那么只有请求 ip-1 的请求会被受理。

```
bind 127.0.0.1 # 注释掉这一行，则监听所有 interface(网卡) 接收到的请求
```

> **注意: **
> bind 的不是请求来源的 IP

## 主从复制
* 配置 Slave 服务器: 只需要在 Slave 服务器的配置文件里加入以下配置

    ```
    slaveof 192.168.1.1 6379 # 指定 Master 的 ip 和端口
    masterauth 123456        # 这是主机的密码
    ```

* Master 的配置不需要修改
* 1 个 Master 可以有多个 Slave

## Redis 类型和命令
> **参考:** [Redis 入门](http://www.hubwiz.com/course/?type=Redis)

* 普通
    * keys pattern
    * exists key
    * expire key seconds
    * ttl key
* String
    * set key value
    * get
    * mset
    * mget
    * setex
    * append
    * strlen
* Hash
    * hset key field value
    * hget key field
    * hmset
    * hmget
    * hgetall key
    * hexists key field
    * hkeys key
    * hlen key
    * hdel key field [field...]
* List
    * lpush
    * rpush
    * lset key index value
    * lpop key
    * lindex key index
    * len
    * ltrim key start stop
* Set (无序)
    * sadd key value
    * smembers key
    * scard key
    * sismember key member
    * srem key member [member...]
* ZSet (有序: score)
    * zadd key score member
    * zrem key member
    * zscore key member
    * zrange key start stop [WITHSCORES]
    * zcard key
    * zrank key member
    * zincrby key increment member

## Redis Benchmark
* `redis-benchmark -h 192.168.1.201 -p 6379 -c 100 -n 100000`: 100 个并发连接，100000 个请求，检测 host 为 192.168.1.201 端口为 6379 的 Redis 服务器性能
* `redis-benchmark -h 192.168.1.201 -p 6379 -q -d 100`: 测试存取大小为100字节的数据包的性能
* `redis-benchmark -t set,lpush -n 100000 -q`: 只测试某些操作的性能
* `redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"`: 只测试某些数值存取的性能

> **Usage:** `redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]`

```
 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 10000)
 -d <size>          Data size of SET/GET value in bytes (default 2)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
                    Using this option the benchmark will get/set keys
                    in the form mykey_rand:000000012456 instead of constant
                    keys, the <keyspacelen> argument determines the max
                    number of values for the random number. For instance
                    if set to 10 only rand:000000000000 - rand:000000000009
                    range will be allowed.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma-separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
```

## 查看有多少个 key
* `dbsize`: 所有 key 的个数，包含过期未删除的 key

    ```
    (integer) 8
    ```
* `info keyspace`: 所有 key 的个数，过期未删除的 key 的个数

    ```
    # Keyspace
    db0:keys=8,expires=0,avg_ttl=0
    ```

## 参考资料
* [Redis 入门](http://www.hubwiz.com/course/?type=Redis)
* [Redis Benchmark](http://my.oschina.net/iepac/blog/705389)
* [Redis 的高级实用特性讲解](http://my.oschina.net/u/2391658/blog/705095)
