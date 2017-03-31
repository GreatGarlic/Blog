---
title: 压力测试
date: 2017-03-31 19:39:05
tags: Spring-Web
---
下面介绍使用 Apache 的 `ab` (Apache Benchmark) 简单的进行压力测试

**主要参数:**
* -n: 总请求数
* -c: 并发用户数
* -C: cookie

**测试:** 

```
ab -n1000 -c30 http://localhost:8080/demo
```

**输出：**

```
Biao: /Applications/MAMP/Library/bin $: ab -n1000 -c30 http://localhost:8080/demo
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests

Server Software:        Apache-Coyote/1.1
Server Hostname:        localhost
Server Port:            8080

Document Path:          /demo
Document Length:        62 bytes

Concurrency Level:      30
Time taken for tests:   0.833 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      228000 bytes
HTML transferred:       62000 bytes
Requests per second:    1201.04 [#/sec] (mean)
Time per request:       24.978 [ms] (mean)
Time per request:       0.833 [ms] (mean, across all concurrent requests)
Transfer rate:          267.42 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    6   7.5      2      51
Processing:     2   19  14.3     16     100
Waiting:        0   17  13.8     15     100
Total:          4   25  13.8     20     100

Percentage of the requests served within a certain time (ms)
  50%     20
  66%     22
  75%     26
  80%     30
  90%     38
  95%     57
  98%     72
  99%     77
 100%    100 (longest request)
​```
>  更强大的压力测试工具可以使用 JMeter，RunLoader