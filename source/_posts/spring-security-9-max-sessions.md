---
title: 限制同一个账号的登陆用户
date: 2017-08-17 22:20:41
tags: Spring-Security
---

有时希望限制同一个账号同时只能有 1 个用户登陆，通常为后一次登录将使前一次登录失效。Spring Security 的 session-management为我们提供了这种限制:

1. 在 web.xml 中定义监听器 HttpSessionEventPublisher

   ```xml
   <listener>
       <listener-class>org.springframework.security.web.session.HttpSessionEventPublisher</listener-class>
   </listener>
   ```

2. 通过 **concurrency-control** 来限制账号登陆数

   ```xml
   <http auto-config="true">
       ...
       <session-management>
           <concurrency-control max-sessions="1"/>
       </session-management>
   </http>
   ```

   ​