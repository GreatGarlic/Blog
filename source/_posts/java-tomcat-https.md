---
title: Tomcat 启用 https
date: 2016-12-02 10:54:27
tags: [Java, Cas]
---

开发时需要用到 SSO 单点登录，一般都会用 Jasig 的 CAS，而 CAS Server 默认要求它运行的服务器启用 https(CAS Client 不强制要求使用 https)，所以就有必要在 Tomcat 中启用 https 了。下面介绍在开发环境中自己生成 ssl 的证书，并在 Tomcat 里启用 https。

<!--more-->

## 1. 生成 keystore

```java
执行命令:
keytool -genkey -alias xtuer -keyalg RSA -keystore server.keystore

按照提示来:
Enter keystore password: // 输入 keystore 密码: 123456
Re-enter new password:   // 再次输入 keystore 密码: 123456
What is your first and last name?
  [Unknown]:             // 输入域名，如 www.xtuer.com, 注意这里不要写 IP 地址
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

// 在当前目录下可以找到一个新生成的文件: server.keystore
```
> * -alias xtuer: `xtuer` 只是一个名字，可以随意取，不过在使用 keystore 生成 cert 文件时需要这个名字，不要忘了就好
> * 在 hosts 文件中添加: 127.0.0.1 www.xtuer.com，因为生成 keystore 时 first and last name 使用了 www.xtuer.com，使用域名而不是 localhost，是为了在局域网中方便多台机器测试 CAS 的功能，CAS 的后续文章中会涉及到

## 2. 复制 server.keystore

1. 在 `<TOMCAT_HOME>` 目录下新建目录 keystore
2. 拷贝 server.keystore 到目录 `<TOMCAT_HOME>`/keystore

## 3. 修改 server.xml

修改 `<TOMCAT_HOME>/conf/server.xml` 文件，添加 https 的 Connector

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

   > * keystoreFile 指向文件 <TOMCAT_HOME>/keystore/server.keystore 文件  
   > * keystorePass 就是刚才生成 certificate keystore 的密码

## 4. 测试 https
1. 访问 [https://www.xtuer.com:8443/](https://www.xtuer.com:8443/)
2. 提示有不安全的证书，接受证书
3. 如能正确访问，则 Tomcat 启用 https 成功

