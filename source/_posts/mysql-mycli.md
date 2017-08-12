---
title: MySQL 命令行客户端 MyCLI
date: 2017-08-12 08:43:49
tags: DB
---

MyCLI 是一个 MySQL 的命令行客户端，可以实现自动补全（auto-completion）和语法高亮，具体特性如下:

* 智能补全
* SQL 语法高亮显示
* 自动完成输入 SQL关键字以及数据库列表
* `SELECT * FROM <tab>` 只显示表名
* `SELECT * FROM users WHERE <tab>` 只显示列名
* 支持 tab 自动补全
* MySQL 的输出会通过 less 命令进行格式化输出
* 支持 ssl 连接

![](/img/db/mycli.gif)

<!--more-->

## 安装

* Mac: 

  ````
  brew install mycli
  ````

* CentOS:

  ```
  sudo yum install pip
  sudo pip install mycli
  ```

* Ubuntu: 

  ```
  sudo apt-get install mycli
  ```

## 登陆

`mycli -h host -u root` 

## 帮助

`mycli -help`