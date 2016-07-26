---
title: Mac 安装 Mysql 和 Nginx
date: 2016-07-26 22:48:02
tags: [Mac, MySQL]
---

Mac 上安装 MySQL 和 Nginx 有很多种方法，例如安装 MAMP 就可以了，也可以单独下载安装包安装，也可以使用 brew 在终端里安装，下面介绍使用 brew 安装的方法。

<!--more-->

## MySQL 的安装和使用
* 安装 MySQL: `brew install mysql`
* 启动 MySQL:
    * 进入: `cd /usr/local/Cellar/mysql/5.7.13/bin/`
    * 启动: `nohup ./mysqld &`
* 访问 MySQL: `./mysql -u root -p`
    * 默认没有设置密码
    * 修改密码: `ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';`
    * MySQL 5.7 及以上修改密码和低版本的不同，上面的方法是修改 MySQL 5.7 的密码，详细信息请参考 MySQL 手册 <http://dev.mysql.com/doc/refman/5.7/en/default-privileges.html>

## Nginx 的安装和使用
* 安装 Nginx: `brew install nginx`
* 启动 Nginx:
    * 进入: `/usr/local/Cellar/nginx/1.10.1/bin`
    * 启动: `sudo ./nginx`
    * 关闭: `sudo ./nginx -s stop`
* 配置 Nginx: `vi /usr/local/etc/nginx/nginx.conf`
