---
title: 使用 CAS 单点登陆
date: 2016-12-08 14:49:12
tags: [Java, Cas]
---

到此，Tomcat 启用了 https，并且在 Tomcat 里部署了 CAS Server，也了解了 Spring Security 的入门，下面就集成 CAS 到 Spring Security 里实现单点登陆。

<!--more-->

## 1. 导出证书

使用在 [Tomcat 启用 https](/java-tomcat-https/) 一节中生成的 `server.keystore` 文件生成证书 `server.cer` 

```
命令行 cd 进入 server.keystore 所在目录

keytool -export -alias xtuer -file server.crt -keystore server.keystore

输入创建 server.keystore 的密码 123456

生成 server.crt
```

## 2. 导入证书

在使用 CAS 登陆的项目所在的电脑上导入上面生成的证书 `server.cer` (不是 CAS Server 所在电脑，而是 CAS Client 所在电脑)

```
命令行 cd 进入 server.cer 所在目录

sudo keytool -import -alias xtuer -file server.crt -keystore /Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home/jre/lib/security/cacerts

Enter keystore password:      // 输入创建 server.keystore 的密码 123456
Re-enter new password:        // 再次输入 123456
Trust this certificate? [no]: // 输入 yes
```

> 证书导入到 `<JAVA_HOME>`/jre/lib/security/cacerts，替换为自己电脑上对应的目录 
>
> 删除已有证书: keytool -delete -alias xtuer -keystore `<JAVA_HOME>`/jre/lib/security/cacerts

## 3. 注册使用 CAS 的站点

修改 `<cas>`/WEB-INF/classes/services/Apereo-10000002.json

```
"serviceId" : "^https://www.apereo.org" 
修改为 
"serviceId" : "^http.*"
```

## 4. 修改 spring-security.xml

详细配置说明请参考 <http://docs.spring.io/spring-security/site/docs/current/reference/html/cas.html>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:security="http://www.springframework.org/schema/security"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd">
    <security:http auto-config="true" entry-point-ref="casEntryPoint">
        <security:custom-filter position="CAS_FILTER" ref="casFilter"/>
        <security:intercept-url pattern="/admin" access="hasRole('ADMIN')"/>
    </security:http>
  
    <security:authentication-manager alias="authenticationManager">
        <security:authentication-provider ref="casAuthenticationProvider"/>
    </security:authentication-manager>

    <bean id="serviceProperties" class="org.springframework.security.cas.ServiceProperties">
        <!-- 此处为当前项目的网址 -->
        <property name="service" value="http://www.xtuer.com:8081/login/cas"/>
        <property name="sendRenew" value="false"/>
    </bean>
    <bean id="casFilter" class="org.springframework.security.cas.web.CasAuthenticationFilter">
        <property name="authenticationManager" ref="authenticationManager"/>
    </bean>

    <bean id="casEntryPoint" class="org.springframework.security.cas.web.CasAuthenticationEntryPoint">
      	<!-- 此处为 CAS 登陆的网址 -->
        <property name="loginUrl" value="https://www.xtuer.com:8443/cas/login"/>
        <property name="serviceProperties" ref="serviceProperties"/>
    </bean>

    <bean id="casAuthenticationProvider" class="org.springframework.security.cas.authentication.CasAuthenticationProvider">
        <property name="authenticationUserDetailsService">
            <bean class="org.springframework.security.core.userdetails.UserDetailsByNameServiceWrapper">
                <constructor-arg ref="userService"/>
            </bean>
        </property>
        <property name="serviceProperties" ref="serviceProperties"/>
        <property name="ticketValidator">
            <bean class="org.jasig.cas.client.validation.Cas20ServiceTicketValidator">
                <!-- 此处为 CAS 的网址 -->
                <constructor-arg index="0" value="https://www.xtuer.com:8443/cas"/>
            </bean>
        </property>
        <property name="key" value="an_id_for_this_auth_provider_only"/>
    </bean>

    <security:user-service id="userService">
        <!-- 这里的密码没有什么用，可以随便填，但是角色 authorities 是需要的 -->
        <!-- CAS 只管理用户名和密码用于登陆，角色，权限等是各子系统自己管理-->
        <!-- 如果 CAS 管理权限等，那么 CAS 需要提供接口，userService 使用接口获取权限 -->
        <security:user name="admin" password="----" authorities="ROLE_ADMIN"/>
        <security:user name="alice" password="----" authorities="ROLE_USER"/>
    </security:user-service>
</beans>
```

## 5. admin.htm

```html
<html>

<body>
    <h1>Title : ${title}</h1>
    <h1>Message : ${message}</h1>
    <#if username??>
        <h2>Welcome : ${username} <a href="https://www.xtuer.com:8443/cas/logout">Logout</a></h2>
    </#if>
</body>

</html>
```

## 6. 登陆测试

1. 访问 [http://www.xtuer.com:8081/admin](http://www.xtuer.com:8081/admin)
2. 被自动重定向到 CAS 进行登陆，输入用户名 admin，密码 admin
3. 登陆成功自动重定向到 <http://www.xtuer.com:8081/admin>
4. 把项目复制一份，修改其使用的端口为 8082，启动项目
5. 访问 <http://www.xtuer.com:8082/admin>，哇，没有登陆就能访问，说明单点登陆成功
6. 点击 `Logout` 注销，另一个子系统的 CAS 登陆状态也被注销了，但是注销后的页面却是停留在 CAS 系统上，后续再介绍单点注销的功能

## 可能遇到的错误

* **Application Not Authorized to Use CAS**  
  The application you attempted to authenticate to is not authorized to use CAS.

  > 解决方法: `<cas>`/WEB-INF/classes/services/Apereo-10000002.json 中修改  
  > `"serviceId" : "^https://www.apereo.org"`   
  > 为  
  > `"serviceId" : "^http.*"`

* PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target

  > 解决方法: 如果没有导入证书，使用 CAS 登陆成功后提示上面的错误，参考第 2 步 `导入证书`

* keytool error: java.io.IOException: Keystore was tampered with, or password was incorrect

  > 解决方法: 上面的错误有可能发生在导入证书的时候，删除 `<JAVA_HOME>`/jre/lib/security/cacerts 然后再次导入证书

## 参考资料

* [TOMCAT 6 中配置 HTTPS](http://www.cnblogs.com/yuanermen/archive/2011/03/30/2000153.html)
* [CAS 单点登录整合 Spring Security](http://blog.csdn.net/ang_dd/article/details/12686901)
