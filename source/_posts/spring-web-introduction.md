---
title: Web 开发简介
date: 2016-10-15 09:25:55
tags: Spring-Web
---
# Web 开发简介

`目的`：使用当前比较流行的技术开发一个具有完整功能的网站。

## 使用的技术
* Web Server: Tomcat
* Web 框架: SpringMVC
* 显示层: Freemarker
* 数据库: MySQL
* 持久层: MyBatis
* 事务: @Transactional
* 日志: Logback
* 风格: RESTful
* 安全: Spring-Security(登录管理，权限管理)
* 项目管理工具: Gradle(自带热加载功能)

<!--more-->

## 开发环境
使用 `IntelliJ IDEA` 和 `Gradle`进行开发。

1. 安装 IntelliJ IDEA 社区版: 
    * [IDEA 介绍](http://www.jetbrains.com/idea/)
    * [IDEA 下载](http://www.jetbrains.com/idea/download/download_thanks.jsp)
2. 安装 Gradle: [Gradle 主页](https://gradle.org)
3. 安装 MySQL: 
    * Mac 下可以安装 `MAMP`
    * Windows 里可以安装 `MAMP`
    * MAMP: 里面集成了数据库 MySQL, MySQL 的网页版管理工具 PhpMySQLAdmin 等
4. 安装 [Tomcat](http://tomcat.apache.org)，开发阶段可以不安装, 而是使用 Gradle  的 Gretty 插件，Gretty 集成了 Jetty 和 Tomcat。

