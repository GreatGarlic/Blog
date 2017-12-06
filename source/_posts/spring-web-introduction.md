---
title: 开发简介
date: 2016-10-15 09:25:55
tags: SpringWeb
---
本系列文章的目的是使用当前比较流行的技术，由浅入深、从入门开始进行介绍，最终开发一个具有完整功能的网站。

## 相关技术

* 后端框架: Spring MVC
* 页面模版: Thymeleaf
* 数据库: MySQL
* 持久层: MyBatis
* 日志框架: Logback
* 架构风格: RESTful
* 访问安全: Spring-Security(登录管理，权限管理)
* 项目管理: Gradle(自带热更新功能)
* 前端框架: Vue、iView

<!--more-->

## 开发环境

使用 `IntelliJ IDEA` 和 `Gradle`进行开发:

1. 安装 IntelliJ IDEA 社区版: 
    * [IDEA 介绍](http://www.jetbrains.com/idea/)
    * [IDEA 下载](http://www.jetbrains.com/idea/download/download_thanks.jsp)
2. 安装 Gradle: [Gradle 主页](https://gradle.org)
3. 安装 MySQL: 
    * Mac 下可以安装 [`MAMP`](https://www.mamp.info)
    * Windows 里可以安装 `MAMP`
    * MAMP: 里面集成了数据库 MySQL, MySQL 的网页版管理工具 PhpMySQLAdmin 等
4. 安装 [Tomcat](http://tomcat.apache.org)，开发阶段不需要安装, 而是使用 Gradle  的 Gretty 插件，Gretty 集成了嵌入式的 Jetty 和 Tomcat


