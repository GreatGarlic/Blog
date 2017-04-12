---
title: 本机安装 ZooKeeper 集群
date: 2017-04-11 20:02:23
tags: [Java, Misc]
---

ZooKeeper 的集群最好是使用 3，5，7，9 奇数台服务器，开发环境可能没有这么多机器给我们使用，不过可以在本机运行多个 ZooKeeper 实例，模拟集群。<!--more-->

## 下载 ZooKeeper

访问 <http://zookeeper.apache.org/releases.html#download> 下载 ZooKeeper。

## 安装 ZooKeeper

ZooKeeper 是绿色软件，解压即是安装。解压下载得到的 **zookeeper-3.3.6.tar.gz** 三次(不要在意版本号)，并重命名得到三个目录，每个目录下是一个 ZooKeeper 的实例，例如放在目录 **/Users/Biao/Documents/zookeeper** 下:

* /Users/Biao/Documents/zookeeper/zookeeper-1
* /Users/Biao/Documents/zookeeper/zookeeper-2
* /Users/Biao/Documents/zookeeper/zookeeper-3

```
├── zookeeper-1
│   ├── bin
│   ├── conf
│   ├── contrib
├── zookeeper-2
│   ├── bin
│   ├── conf
│   ├── contrib
├── zookeeper-3
│   ├── bin
│   ├── conf
│   ├── contrib
```

## data 和 log 目录

为每个 ZooKeeper 实例创建独立的数据存储目录 data 和日志目录 log，例如放在目录 **/Users/Biao/Documents/zookeeper** 下：

* /Users/Biao/Documents/zookeeper/temp/zk1
* /Users/Biao/Documents/zookeeper/temp/zk2
* /Users/Biao/Documents/zookeeper/temp/zk3

```
/Users/Biao/Documents/zookeeper/temp/zk1

├── temp
│   ├── zk1
│   │   ├── data
│   │   └── log
│   ├── zk2
│   │   ├── data
│   │   └── log
│   └── zk3
│       ├── data
│       └── log
```

>  zk1 是 ZooKeeper 实例 1 的目录
>
>  zk2 是 ZooKeeper 实例 2 的目录
>
>  zk3 是 ZooKeeper 实例 3 的目录

## 服务器ID 文件 myid

* 创建文件 **/Users/Biao/Documents/zookeeper/temp/zk1/data/myid**，内容为 **1**
* 创建文件 **/Users/Biao/Documents/zookeeper/temp/zk2/data/myid**，内容为 **2**
* 创建文件 **/Users/Biao/Documents/zookeeper/temp/zk3/data/myid**，内容为 **3**

## 配置文件 zoo.cfg

为每个 ZooKeeper 实例创建配置文件，在 ZooKeeper 的 conf 目录中

* 创建 **/Users/Biao/Documents/zookeeper/zookeeper-1/conf/zoo.cfg**，内容为:

  ```
  tickTime=2000
  initLimit=10
  syncLimit=5
  dataDir=/Users/Biao/Documents/zookeeper/temp/zk1/data
  dataLogDir=/Users/Biao/Documents/zookeeper/temp/zk1/log
  clientPort=2181
  server.1=localhost:2287:3387
  server.2=localhost:2288:3388
  server.3=localhost:2289:3389
  ```
  > clientPort: the port at which the clients will connect，例如下面用到的 **zkCli.sh**

* 创建 **/Users/Biao/Documents/zookeeper/zookeeper-2/conf/zoo.cfg**，内容为:

  ```
  tickTime=2000
  initLimit=10
  syncLimit=5
  dataDir=/Users/Biao/Documents/zookeeper/temp/zk2/data
  dataLogDir=/Users/Biao/Documents/zookeeper/temp/zk2/log
  clientPort=2182
  server.1=localhost:2287:3387
  server.2=localhost:2288:3388
  server.3=localhost:2289:3389
  ```

* 创建 **/Users/Biao/Documents/zookeeper/zookeeper-3/conf/zoo.cfg**，内容为:

  ```
  tickTime=2000
  initLimit=10
  syncLimit=5
  dataDir=/Users/Biao/Documents/zookeeper/temp/zk3/data
  dataLogDir=/Users/Biao/Documents/zookeeper/temp/zk3/log
  clientPort=2183
  server.1=localhost:2287:3387
  server.2=localhost:2288:3388
  server.3=localhost:2289:3389
  ```


> 因为是在一台机器上模拟集群，所以每个 ZooKeeper 实例的端口号 clientPort 不同。
>
> 生产环境中，分布式集群部署的步骤与上面基本相同，只不过因为各 ZooKeeper 分布在不同的机器，上述配置文件中的 localhost 换成各服务器的真实 IP 即可。分布在不同的机器后，不存在端口冲突问题，可以让每个服务器的zk均采用相同的端口，这样管理起来比较方便。

## 启动 ZooKeeper
执行下面的命令启动 ZooKeeper 服务器:

* /Users/Biao/Documents/zookeeper/zookeeper-1/bin/zkServer.sh start
* /Users/Biao/Documents/zookeeper/zookeeper-2/bin/zkServer.sh start
* /Users/Biao/Documents/zookeeper/zookeeper-3/bin/zkServer.sh start

使用命令 **jps** 可以看到 ZooKeeper 的进程信息

```
17145 QuorumPeerMain
17212 QuorumPeerMain
17234 QuorumPeerMain
```

或则从任务管理器里可以看到 3 个名为 java 的进程，看看进程信息就知道是不是 ZooKeeper 的进程了，很多 Java 程序的进程名都叫 java。

## 访问 ZooKeeper

执行命令 `bin/zkCli.sh -server localhost:2181` 使用 ZooKeeper 客户端访问 ZooKeeper，连接成功则说明 ZooKeeper 服务启动了。

```
Connecting to localhost:2181
2017-04-11 21:33:56,163 - INFO  [main:Environment@97] - Client environment:zookeeper.version=3.3.6-1366786, built on 07/29/2012 06:22 GMT
2017-04-11 21:33:56,165 - INFO  [main:Environment@97] - Client environment:host.name=192.168.0.101
2017-04-11 21:33:56,166 - INFO  [main:Environment@97] - Client environment:java.version=1.8.0_77
2017-04-11 21:33:56,166 - INFO  [main:Environment@97] - Client environment:java.vendor=Oracle Corporation
...
```

至此，本机 ZooKeeper 的集群搭建完成，以后就可以在此基础上使用 ZooKeeper 开发了，例如使用 Dubbo 开发分布式服务。