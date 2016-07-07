---
title: Java 发邮件
date: 2016-07-07 09:46:54
tags: [Java, Util]
---

可以使用 `Apache Commons Mail` 或者 `Spring Mail` 发送邮件。

Spring 项目中推荐使用 Spring Mail 发送邮件，非 Spring 项目里可以使用 Apache Commons Mail 发邮件 (需要的 Jar 包相对少一些)。

<!--more-->

## Apache Commons Mail
### Gradle 依赖
```groovy
compile('org.apache.commons:commons-email:1.4')
compile('javax.mail:'javax.mail-api:1.5.5')
compile('com.sun.mail:javax.mail:1.5.5')
```

### 发送邮件
```java
import org.apache.commons.mail.DefaultAuthenticator;
import org.apache.commons.mail.Email;
import org.apache.commons.mail.EmailException;
import org.apache.commons.mail.SimpleEmail;
```

```java
public void sendCommonsMail() throws EmailException {
    Email email = new SimpleEmail();
    email.setHostName("smtp.mail.me.com");
    email.setSmtpPort(587);
    email.setAuthenticator(new DefaultAuthenticator("biao.mac@icloud.com", "***password***"));
    // email.setSSLOnConnect(true);
    email.setStartTLSEnabled(true);
    email.setFrom("biao.mac@icloud.com");
    email.addTo("biao.mac@qq.com");
    email.setSubject("Commons Mail");
    email.setMsg("This is a test mail ... :-)");
    email.send();
}
```

> 这里的 SMTP 使用的是苹果的 CloudMail 的 SMTP  
> 不同的 SMTP 配置不一样，具体要参考发送邮件服务的 SMTP 配置  
> 更多例子请参考: <https://commons.apache.org/proper/commons-email/userguide.html>


## Spring Mail
### Gradle 依赖
```groovy
compile('org.springframework:spring-context-support:4.3.0.RELEASE')
compile('javax.mail:'javax.mail-api:1.5.5')
compile('com.sun.mail:javax.mail:1.5.5')
testCompile('junit:junit:4.12')
testCompile('org.springframework:spring-test:4.3.0.RELEASE')
```

### Spring Bean 配置: `spring-mail.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <property name="host" value="smtp.mail.me.com" />
        <property name="port" value="587" />
        <property name="username" value="biao.mac@icloud.com" />
        <property name="password" value="***password***" />

        <property name="javaMailProperties">
            <props>
                <prop key="mail.smtp.auth">true</prop>
                <prop key="mail.smtp.starttls.enable">true</prop>
            </props>
        </property>
    </bean>

    <bean id="mailService" class="com.xtuer.service.MailService">
        <property name="mailSender" ref="mailSender" />
    </bean>
</beans>
```

> 这里的 SMTP 使用的是苹果的 CloudMail 的 SMTP  
> 不同的 SMTP 配置不一样，具体要参考发送邮件服务的 SMTP 配置

### MailService

```java
package com.xtuer.service;

import org.springframework.mail.MailSender;
import org.springframework.mail.SimpleMailMessage;

public class MailService {
    private MailSender mailSender;

    public void setMailSender(MailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void sendMail(String from, String to, String subject, String msg) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(msg);
        mailSender.send(message);
    }
}
```

### 发送邮件
```java
import com.xtuer.service.MailService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;

@RunWith(SpringRunner.class)
@ContextConfiguration({"classpath:spring-mail.xml"})
public class TestMail {
    @Resource(name="mailService")
    private MailService mailService;

    // 使用 Spring Mail 发送邮件
    @Test
    public void sendSpringMail() {
        mailService.sendMail("biao.mac@icloud.com", "biao.mac@qq.com", "Spring Mail", "This is only for test!");
    }
```
