---
title: CentOS 7 简单使用
date: 2017-08-01 10:49:37
tags: Mac
---

## 下载 CentOS

访问  <https://www.centos.org/download/> 下载 **Minimal ISO** 即可，其他的虽然功能齐全，但是太大了。

## 必要工具

* yum install zip unzip: 

  ```
  # 把文件夹 H5 和文件 x.html 压缩成 result.zip
  zip -r result.zip H5 x.html

  # 解压 filename.zip, 如无 -d 则解压到当前目录，有则解压到目录 dest-directory
  unzip filename.zip [-d dest-directory]
  ```

* yun install bzip2 (解压 .bz2 文件)

* yum install net-tools (安装后才能使用 ifconfig 等)

* tar 解压 tar.gz: `tar xf filename.tar.gz` 

* 安装 tree: `yum install tree`

* 安装 7z: `yum install -y p7zip`，如果不能用 yum 安装，可以自己编译

  ```
  wget http://nchc.dl.sourceforge.net/project/p7zip/p7zip/9.20.1/p7zip_9.20.1_src_all.tar.bz2
  tar -jxvf p7zip_9.20.1_src_all.tar.bz2
  cd p7zip_9.20.1
  make
  make install
  ```

  <!--more-->

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

## 更换 yum 镜像

国内访问 yum 默认镜像太慢，可以更换为比较稳定的阿里云镜像。

1. 备份:

   ```
   mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
   ```

2. 下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/

   * CentOS 6

     ```
     wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
     ```

   * CentOS 7

     ```
     wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
     ```

3. 运行 `yum makecache` 生成缓存

## 7z 使用

```
# 解压 filename.7z 到当前目录
7za x filename.7z

# 解压 filename.7z 到 here 目录
7za x filename.7z -ohere

命令: 7za {a|d|l|e|u|x} 压缩包文件名 {文件列表或目录，可选}
x: 以完整路径解压
a: 向压缩包里添加文件或创建压缩包，如向 001.7z 添加 001.jpg，执行：7za a 001.7z 001.jpg；
   将001目录打包执行：7za a 001.7z 001；
d: 从压缩里删除文件，如将 001.7z 里的 001.jpg 删除，执行：7za d 001.7z 001.jpg
l: 列出压缩包里的文件，如列出 001.7z 里的文件，执行：7za l 001.7z
e: 解压到当前目录，目录结构会被破坏，如 001.rar 内有如下目录及文件 123/456/789.html，
   执行：7za e 001.rar，目录 123 和 456 及文件 789.html 都会存放在当前目录下。
```

## 安装 Nodejs 8

直接 `yum install nodejs` 安装的是 Nodejs 6，如果想要安装 Nodejs 8、Nodejs 9 则需要看[官方文档](https://www.hugeserver.com/kb/install-nodejs8-centos7-debian8-ubuntu16/)，安装步骤如下

```
curl -sL https://rpm.nodesource.com/setup_8.x | bash -
yum install nodejs

# 安装后查看版本
node -v
```

## 安装 MongoDB

详细内容请参考[官方文档](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/#configure-the-package-management-system-yum)，安装步骤如下:

1. 指定 MongoDB 的原: 创建 `/etc/yum.repos.d/mongodb-org-3.6.repo`

   ```
   [mongodb-org-3.6]
   name=MongoDB Repository
   baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
   gpgcheck=1
   enabled=1
   gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
   ```

2. 安装 MongoDB: `sudo yum install -y mongodb-org`

3. 启动 MongoDB: `sudo service mongod start`

4. 关闭 MongoDB: `sudo service mongod stop`

5. 访问 MongoDB: `mongo --host 127.0.0.1:27017`

MongoDB 默认使用端口 27017，配置在 `/etc/mongod.conf`。