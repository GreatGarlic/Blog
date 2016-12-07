---
title: 安装 CAS-Server
date: 2016-12-02 15:16:57
tags: [Java, Cas]
---

CAS 是中央认证服务 Central Authentication Service 的简称，最初由耶鲁大学的 Shawn Bayern 开发，后由 Jasig 社区维护，经过十多年发展，目前已成为影响最大、广泛使用的、基于 Java 实现的、开源 SSO 解决方案。

2012 年，Jasig 和另一个有影响的组织 Sakai Foundation 合并，组成 Apereo。Apereo 是一个由高等学术教育机构发起组织的联盟，旨在为学术教育机构提供高质量软件，当然很多软件也被大量应用于商业环境，譬如 CAS。目前 CAS 由 Apereo 社区维护。

CAS 的安装并不是很容易，因为官网不再提供二进制包下载，需要我们下载 CAS 源码并自己编译，最新版的 CAS 使用 Gradle 编译，下面介绍 CAS 的安装 <!--more-->

1. 下载 CAS
   1. 访问 [https://www.apereo.org/projects/cas/download-cas](https://www.apereo.org/projects/cas/download-cas)
   2. 点击要下载的 CAS 版本， `CAS v4.2.1`
   3. 在打开的页面下载 CAS 的源码
2. 解压 CAS 源码，得到目录 `cas-4.2.1`
3. 命令进入 CAS 源码目录 `cas-4.2.1`
4. 执行命令 `gradle :cas-server-webapp:build` 编译 CAS 的 webapp
5. 编译结束后得到 `cas-4.2.1/cas-server-webapp/build/libs/cas-server-webapp-4.2.1.war`
6. 复制 `cas-server-webapp-4.2.1.war` 到 `<Tomcat>/webapps` 下并重命名为 `cas.war`
7. CAS 默认需要使用 https，Tomcat 启用 https，请参考  [Tomcat 启用 https](/java-tomcat-https/)
8. 启动 Tomcat
9. 访问 [https://localhost:8443/cas](https://localhost:8443/cas)
10. 输入用户名 casuser，密码 Mellon
11. 登陆成功

> CAS 默认的用户名密码 casuser/Mellon 是不是不好记？没关系，可以修改 `cas/WEB-INF/cas.properties` 中的 `accept.authn.users` 为你想要的，例如修改为 
>
> * accept.authn.users=admin::admin
> * accept.authn.users=admin::Passw0rd,alice::Passw0rd

到此，CAS Server 安装成功。

CAS 的认证逻辑如图  
![](/img/cas/cas.png)

## 参考资料

* [CAS4.0 环境的搭建](http://www.cnblogs.com/vhua/p/cas_1.html)
* [轻松玩转 SSO CAS](http://www.imooc.com/article/3576)

