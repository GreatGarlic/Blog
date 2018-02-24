---
title: MongoDB 初步接触
date: 2018-02-11 22:26:51
tags: Mac
---

## 下载安装

不同的系统安装 MongoDB 差别挺大的:

* Mac: 使用 `brew install mongodb`
* Linux: 参考 <http://qtdebug.com/mac-centos7/#安装-MongoDB>
* Windows: [下载](http://www.mongodb.org/downloads)然后安装，安装的时候不要选择安装 MongoDB Compass，因为需要联网下载，国内有可能很久都装不好

## 启动访问

* 启动 MongoDB:
  * `mongod`

  * `mongod --config C:/etc/mongod.conf`

    一般 Windows 才会手动指定配置文件，Linux 和 Mac 都会使用默认的配置文件，下面有介绍
* 访问 MongoDB: 
  * `mongo`
  * `mongo --host IP` <!--more-->

## 配置文件

```yaml
systemLog:
    destination: file
    path: D:/MongoDB/logs/mongodb.log #日志输出文件路径
    logAppend: true
storage:
    dbPath: D:/MongoDB/data #数据库路径
net:
    bindIp: 0.0.0.0 #允许其他电脑访问
```

> 默认只允许本机访问，在 mongod.conf 中配置 bindIp 允许其他机器访问: 
>
> * `bindIp = 127.0.0.1, 192.168.1.10`: 使用这 2 个 IP 访问
> * `bindIp = 0.0.0.0`: 任意机器
>
> 注意：Winows 中 MongoDB 的数据库目录需要我们手动创建，如果目录不存在则会启动失败
>
> * 默认的目录为 `C:/data/db`
> * 上面配置文件的为 `D:/MongoDB/data` 和 `D:/MongoDB/logs`。

配置文件路径:

* Linux: `/etc/mongod.conf`，自动创建，启动时自动使用

* Mac: `/usr/local/etc/mongod.conf`，自动创建，启动时自动使用

* Windows: 手动创建并使用 `--config` 参数指定

  Windows 安装 MongoDB 后不会自动生成配置文件，如果要使用，需要我们自己创建，并在启动时使用参数 `--config` 指定

## 信息

* `help`: 查看 MongoDB 支持哪些命令

* `db.help()`: 查看当前数据库支持哪些的方法

  ```
  db.getName() - 当前 DB 的名字
  db.stats() - 当前 DB 的状态，例如名字，有多少 collections，数据库大小等
  db.dropDatabase() - 删除当前数据库
  ```


* `db.collection_name.help()`: DBCollection help，例如各种 CURD 命令

  ```
  db.test.drop() - drop the collection 删除 collection test
  db.test.dropIndexes()
  db.test.find(...).count()
  ```

* `show dbs`: 显示所有数据库

* `use db_name`: 使用指定数据库

* `show collections`: 显示当前数据库下的所有 collection

## 查询

* `db.collection_name.find()`: 显示 collection_name 下的所有 document

  > `db` 是命令的前缀，CRUD 时需要带上。
