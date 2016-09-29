---
title: 常用命令
date: 2016-06-29 16:27:55
tags: Misc
---

各种常用的命令

<!--more-->

| 命令                  | 说明                  |
| -------------------- | -------------------- |
| mvn clean            |                      |
| mvn compile          |                      |
| mvn package          | Maven 打包            |
| mvn tomcat7:run      | 启动嵌入式 Tomcat      |
| mvn install          | 安装 jar 包到本地仓库    |
| activemq start       | 启动 ActiveMQ         |
| activemq stop        | 关闭 ActiveMQ         |
| redis-server         | 启动 Redis 服务器 |
| redis-cli            | 打开 Redis 客户端，输入命令，例如 `keys *` |
| gradle build         | 使用 Gradle 构建项目    |
| gradle tasks         | 列出所有的 task        |
| gradle jettyRun      | 启动嵌入式 Jetty，Spring 不会打印日志 |
| gradle -i jettyRun   | 启动嵌入式 Jetty，Spring 会打印日志 |
| gradle -i appStart   | 启动嵌入式 Greety 的 Jetty 或则 Tomcat |
| groovy hello.groovy  | 运行 Groovy 脚本       |
| ab -n1000 -c30 http://localhost:8080/demo | 压力测试，本机测试，可以减少网络访问的延迟 |
| brew install gradle  | 安装 Gradle           |
| brew install activemq| 安装 ActiveMQ         |
| brew install redis   | 安装 Redis            |
| brew install groovy  | 安装 Groovy           |
| brew install maven   | 安装 Maven            |
| brew install ant     | 安装 Ant              |
| brew install tomcat  | 安装 Tomcat           |
| brew uninstall comcat| 卸载 Tomcat           |
| show full processlist | MySQL 显示连接        |
| lsof -n -P | 查看使用端口的进程 ID，查看 8080 端口的使用情况: lsof -n -P \| grep :8080 |
