---
title: CentOS 7 简单使用
date: 2017-08-01 10:49:37
tags: Mac
---

## 下载 CentOS

访问  <https://www.centos.org/download/> 下载 **Minimal ISO** 即可，其他的虽然功能齐全，但是太大了。

## 必要工具

* yum install zip unzip: `unzip filename.zip [-d dest-directory]`
* yum install net-tools (安装后才能使用 ifconfig 等)
* tar 解压 tar.gz: `tar xf filename.tar.gz` <!--more-->

## 安装 Java

* 下载 Linux 64  位版本的 [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

  > 查看系统的信息 uname -a，可以看到 CentOS 7 是 64 位的
  >
  > Linux bogon 3.10.0-514.el7.x86\_64 #1 SMP Tue Nov 22 16:42:41 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux

* 解压到 **/usr/local** 目录

* 修改 **.bash_profile** 文件(在用户目录下)，添加 Java 到环境变量

  > export JAVA_HOME="/usr/local/jdk1.8.0_144"
  >
  > PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
  >
  > export PATH

* 使修改的 **.bash_profile** 生效: `source .bash_profile`

* 执行 `java -version` 正常输出则安装好了

## 安装 Tomcat

* 下载 Tomcat 8
* 解压到 **/usr/local** 目录
* 运行 Tomcat: `<tomcat>/bin/startup.sh`
* wget 访问 Tomcat 首页: `wget http://127.0.0.1:8080` 
* 其他机器访问 Tomcat 首页，拒绝访问，因为 CentOS 7 的端口 8080 默认是禁用的，需要开放此端口

## 开放端口

CentOS 7 使用 firewalld 控制端口，不再使用 iptables，默认 8080 端口等都是禁用的。

* 开启端口: `firewall-cmd --zone=public --add-port=8080/tcp --permanent`

  > --zone #作用域
  > --add-port=8080/tcp  #添加端口，格式为：端口/通讯协议
  > --permanent  #永久生效，没有此参数重启后失效

* 重启防火墙: `firewall-cmd --reload`

* 查看打开的端口: `firewall-cmd --zone=public --list-ports`

* 正在监听的端口: `netstat -an | grep LISTEN` 

参考: [CentOS 7 firewalld 使用简介](http://blog.csdn.net/spxfzc/article/details/39645133)

## SSH 登陆

`ssh root@192.168.82.130`，然后输入密码。

> ssh username@host

## SSH + Key 登陆

* 客户端: 
  1. 执行 `ssh-keygen -t rsa -C "your_email@example.com"`
  2. 在用户目录的 **.ssh** 目录下的到 **id_rsa** 和 **id_rsa.pub**
  3. 复制 **id_rsa.pub** 到 CentOS 的服务器上
* 服务器:
  1. 用户的 **id_rsa.pub** 放到 **/root** 目录
  2. `mkdir /root/.ssh`
  3. `cd /root/.ssh`
  4. `cat /root/id_rsa.pub >> /root/.ssh/authorized_keys`
* 客户端: `ssh username@host`，不需要密码，直接能访问了