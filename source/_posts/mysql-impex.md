---
title: MySQL 导入导出 SQL 文件
date: 2017-11-28 16:02:25
tags: DB
---

## 创建数据库

```
CREATE DATABASE IF NOT EXISTS databaseName DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

## 导入 SQL 文件

MySQL 可以使用 GUI 客户端导入 SQL 文件，此外在命令行下有常用下面 2 种方式导入 SQL 文件(先要创建好数据库)

* 使用 **mysql** 导入
  1. `./mysql -u root -p 数据库名 < 导入的文件名.sql`
  2. 输入密码
* 使用 **source** 导入
  1. `./mysql -u root -p`
  2. 输入密码
  3. `use databaseName`
  4. `source 导入的文件名.sql`

## 导出 SQL 文件

导出有 2 种: 导出表结构，导出数据库(包含表结构和表中的数据)

* 导出表结构: `mysqldump -u username -p 数据库名 > 导出的文件名.sql`
* 导出数据库: `mysqldump -u username -p -d 数据库名 > 导出的文件名.sql`