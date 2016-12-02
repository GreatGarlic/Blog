---
title: Tomcat 启用 https
date: 2016-12-02 10:54:27
tags: Java
---

开发时需要用到 SSO 单点登录，一般都会用 Jasig 的 CAS，而 CAS 4.0 默认要求使用 https 才能访问，所以就有必要在 Tomcat 中启用 https 了。下面介绍在开发环境中自己生成 ssh 的证书，并在 Tomcat 里启用 https。

<!--more-->

1. 生成 keyStore

   ```java
   执行命令:
   keytool -genkey -alias mykey -keyalg RSA -keystore server.keystore

   按照提示来:
   Enter keystore password: // 输入 keystore 密码: 123456
   Re-enter new password:   // 再次输入 keystore 密码: 123456
   What is your first and last name?
     [Unknown]:             // 输入主机名, 如果是本机就写 localhost, 注意这里不要写 IP 地址
   What is the name of your organizational unit?
     [Unknown]:             // 输入单位名称 xtuer
   What is the name of your organization?
     [Unknown]:             // 输入组织名称 xtuer
   What is the name of your City or Locality?
     [Unknown]:             // 输入城市或区域名称 beijing
   What is the name of your State or Province?
     [Unknown]:             // 输入州或省份名称 beijing
   What is the two-letter country code for this unit?
     [Unknown]:             // 输入单位所在国家的两字母国家代码 cn

   Is CN=localhost, OU=xtuer, O=xtuer, L=beijing, ST=beijing, C=cn correct?
     [no]:                  // 检查填写的信息, 如果对了, 输入 yes 并按下回车

   // 输入<mykey>的主密码, 如果和 keystore 密码相同，按回车
   Enter key password for <mykey>
   	(RETURN if same as keystore password):  

   // 在当前目录下可以找到一个文件: server.keystore，其中就包含了自签名的证书
   ```

2. 在 `<TOMCAT_HOME>` 目录下新建目录 keystore，并拷贝 server.keystore 到其目录下

3. 修改 `<TOMCAT_HOME>/conf/server.xml` 文件，添加 https 的 Connector

   ```xml
   <Connector port="8443"
          minSpareThreads="5"
          maxSpareThreads="75"
          enableLookups="true"
          disableUploadTimeout="true"
          acceptCount="100"
          maxThreads="200"
          scheme="https"
          secure="true"
          SSLEnabled="true"
          clientAuth="false"
          sslProtocol="TLS"
          keystoreFile="keystore/server.keystore"
          keystorePass="123456"
   />
   ```

   > keystoreFile 指向文件 <TOMCAT_HOME>/keystore/server.keystore 文件  
   > keystorePass 就是刚才生成 certificate keystore 的密码

4. 测试 https: 访问 [https://localhost:8443/](https://localhost:8443/)，提示有不安全的证书，接受证书，并能正确访问首页，则 Tomcat 启用 https 成功

   ​
