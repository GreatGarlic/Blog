---
title: Spring Security Session + Token 认证
date: 2017-08-21 22:08:53
tags: Spring-Security
---

前面通过表单进行登陆，会为用户创建一个 session 保存在服务器端，session id 保存在 cookie 中，每次访问服务器的时候服务器端从 cookie 中读取 session id 然后找到用户的 session，就能知道当前用户的信息。但是对于移动端来说，传递 cookie 不是很方便，一般都会使用 token 来进行验证。

Token 就是一个字符串(可以使用 uuid)，验证时使用的 token 可以理解为和 session id 的功能差不多:

1. 用户申请 token 时，可以把 token 作为 key，用户信息的对象作为 value 保存到 Redis 中，把 token 返回给移动端
2. 移动端保存 token，有很多种方式，例如保存到文件中，sqlite 里都可以
3. 每次访问的时候把 token 放到请求的 header 中
4. 服务器端从 header 中读取 token，然后用 token 作为 key 去 Redis 中去读用户数据
5. 如果读取到的用户数据有效，则说明用户是合法的，认证通过，继续访问，否则返回错误，终止请求

使用纯 token 验证，不支持 session，这样的应用一般都是用来提供纯数据服务(应用中没有网页，很多微服务就是这样的)，以下叫 **DSA**(Data Service Application)，但是数据也是需要后台功能来管理的，大多都会使用 Web 应用，叫 **DMA**(Data Management Application)，Web 应用需要使用 session，也就是说 DSA 和 DMA 是独立的 2 个应用，不能共存，因为 DSA 中不支持 session，而 DMA 中需要 session。这种设计的好处是 DSA 很轻量级，只关心数据服务，能够降低开发的复杂度，还有其它比如每个服务都很简单，只关注于一个业务功能，每个微服务可以由不同的团队独立开发，微服务是松散耦合的等等。但是也有缺点，比如有可能对资源的访问需要重复实现，例如一个电子图书馆程序，读取图书信息的 API **/api/books/{bookId}** 在 DSA 中需要实现，在 DMA 中也要提供实现，因为 DMA 中也需要读取图书信息进行管理，就算用分布式服务使用 dubbo 负责服务治理，由 DMA 提供访问数据的逻辑，但是 DSA 和 DMA 里都至少也要各自有个 Controller 来处理这个 URL 吧。

本文的目的，是要实现一个 Web 应用即支持 session，同时又能支持使用 token 进行身份验证时不生成 session:

* 浏览器访问 /api/books/{bookId} 时，从 cookie 中读取 session id 找到对应的 session，获取当前用户，如果没有登陆则跳转到登陆页面进行登陆，登陆成功会创建 session
* 移动端访问 /api/books/{bookId} 时，从 header 中读取 token 找到对应的用户，如果没有 token 或者 token 过期、用户信息无效则返回错误提示未登陆认证(token 可以事先请求保存起来)，整个过程不会产生 session <!--more-->

Spring Security 中已经提供了表单登陆认证的功能，如下配置：

```xml
<http auto-config="true">
    <intercept-url pattern="/page/admin" access="hasRole('ROLE_ADMIN')"/>
    <intercept-url pattern="/api/json" access="hasAnyRole('ROLE_ADMIN', 'ROLE_USER')"/>

    <form-login login-page="/page/login"
                login-processing-url="/login"
                default-target-url  ="/"
                authentication-failure-url="/page/login?error=1"
                username-parameter="username"
                password-parameter="password"/>
</http>
```

> form-login 使用 UsernamePasswordAuthenticationFilter 作为 filter 进行身份验证，而且这个 filter 是不能被替换掉的。

要支持 token 的认证，需要实现一个 filter，继承 AbstractAuthenticationProcessingFilter，然后插入到 FORM_LOGIN_FILTER 的前面:

```xml
<custom-filter ref="tokenAuthenticationFilter" before="FORM_LOGIN_FILTER"/>
```

> FORM_LOGIN_FILTER 就是 UsernamePasswordAuthenticationFilter。

下面的类 TokenAuthenticationFilter 用于 token 的身份认证，XHttpSessionSecurityContextRepository 确定是否创建 session，在 spring-security.xml 中进行配置。

## TokenAuthenticationFilter

当 TokenAuthenticationFilter.doFilter 发现 header 中有 auth-token 时则使用 auth-token 去查找用户信息，如果找不到或者用户信息无效则认证失败，直接返回 SC_UNAUTHORIZED，请求终止。如果查找到有效的用户信息则认证成功，chain.doFilter 会调用下一个 filter UsernamePasswordAuthenticationFilter，它的 doFilter 中 requiresAuthentication(request, response) 返回 false，不会再继续认证操作，而是继续调用下一个 filter。

关键点是，TokenAuthenticationFilter.doFilter() 中为了不让 HttpSessionSecurityContextRepository 在使用 token 认证时创建 session，需要调用 allowSessionCreation.set(false)，在确定是否创建 session 时检查 TokenAuthenticationFilter.isAllowSessionCreation()。

> AbstractAuthenticationProcessingFilter 也有属性 allowSessionCreation，但是设置了是没有用的。

```java
package com.xtuer.security;

import com.xtuer.bean.User;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * 使用 token 进行身份验证的过滤器。
 * 如果 request header 中有 auth-token，使用 auth-token 的值查询对应的登陆用户，如果用户有效则放行访问，否则返回 401 错误。
 */
public class TokenAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    private static ThreadLocal<Boolean> allowSessionCreation = new ThreadLocal<>(); // 是否允许当前请求创建 session

    public TokenAuthenticationFilter() {
        super(new AntPathRequestMatcher("/login", "POST")); // 参考 UsernamePasswordAuthenticationFilter
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException, IOException, ServletException {
        String token = request.getHeader("auth-token");

        // 模拟 token 无效返回 null
        if (!"123".equals(token)) {
            return null;
        }

        // 使用 token 信息查找缓存中的登陆用户信息，下面为了测试方便直接写死一个
        User user = new User("admin", "不重要", "ROLE_ADMIN");

        return new UsernamePasswordAuthenticationToken(user, user.getPassword(), user.getAuthorities());
    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
        Authentication auth = null;

        // 默认创建 session
        allowSessionCreation.set(true);

        // 如果 header 里有 auth-token 时，则使用 token 查询用户数据进行登陆验证
        if (request.getHeader("auth-token") != null) {
            // 1. 尝试进行身份认证
            // 2. 如果用户无效，则返回 401
            // 3. 如果用户有效，则保存到 SecurityContext 中，供本次方式后续使用
            auth = attemptAuthentication(request, response);

            if (auth == null) {
                response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Token 无效，请重新申请 token");
                return;
            }

            // 保存认证信息到 SecurityContext，禁止 HttpSessionSecurityContextRepository 创建 session
            allowSessionCreation.set(false);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }

        // 继续调用下一个 filter: UsernamePasswordAuthenticationToken
        chain.doFilter(request, response);
    }

    public static boolean isAllowSessionCreation() {
        return allowSessionCreation.get();
    }
}
```

> 没有校验 token，attemptAuthentication 直接返回了一个 authentication 只是为了测试方便，实际项目中要根据具体情况进行实现，这里就不再赘述。

## XHttpSessionSecurityContextRepository

XHttpSessionSecurityContextRepository 的代码完全复制 HttpSessionSecurityContextRepository，然后修改了创建 session 的函数 createNewSessionIfAllowed():

```java
package org.springframework.security.web.context;

public class XHttpSessionSecurityContextRepository implements SecurityContextRepository {
    ...
    
    private HttpSession createNewSessionIfAllowed(SecurityContext context) {
        ...

        if (!allowSessionCreation || !TokenAuthenticationFilter.isAllowSessionCreation()) {
            if (logger.isDebugEnabled()) {
                logger.debug("The HttpSession is currently null, and the "
                        + HttpSessionSecurityContextRepository.class.getSimpleName()
                        + " is prohibited from creating an HttpSession "
                        + "(because the allowSessionCreation property is false) - SecurityContext thus not "
                        + "stored for next request");
            }

            return null;
        }
        
        ...
    }

    ...
}
```

## spring-security.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans
        xmlns="http://www.springframework.org/schema/security"
        xmlns:beans="http://www.springframework.org/schema/beans"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd">
    <context:annotation-config/>

    <http security="none" pattern="/page/login"/>
    <http security="none" pattern="/static/**"/>

    <http auto-config="true" security-context-repository-ref="sessionSecurityContextRepository">
        <intercept-url pattern="/page/admin" access="hasRole('ROLE_ADMIN')"/>

        <form-login login-page="/page/login"
                    login-processing-url="/login"
                    default-target-url="/"
                    authentication-failure-url="/page/login?error=1"
                    username-parameter="username"
                    password-parameter="password"/>
        <logout logout-url="/logout" logout-success-url="/page/login?logout=1"/>
        <access-denied-handler error-page="/page/deny"/>

        <csrf disabled="true"/>
        <remember-me key="uniqueAndSecret" token-validity-seconds="2592000"/>

        <custom-filter ref="tokenAuthenticationFilter" before="FORM_LOGIN_FILTER"/>
    </http>

    <authentication-manager alias="authenticationManager">
        <authentication-provider user-service-ref="userDetailsService"/>
    </authentication-manager>

    <beans:bean id="tokenAuthenticationFilter" class="com.xtuer.security.TokenAuthenticationFilter">
        <beans:property name="authenticationManager" ref="authenticationManager"/>
    </beans:bean>
    <beans:bean id="userService" class="com.xtuer.service.UserService"/>
    <beans:bean id="userDetailsService" class="com.xtuer.security.UserDetailsService"/>
    <beans:bean id="sessionSecurityContextRepository" class="org.springframework.security.web.context.XHttpSessionSecurityContextRepository"/>
</beans:beans>

```

使用 `security-context-repository-ref` 注入我们自己实现的 HttpSessionSecurityContextRepository，替代默认的实现。

用 `<custom-filter ref="tokenAuthenticationFilter" before="FORM_LOGIN_FILTER"/>` 在 FORM_LOGIN_FILTER 前插入我们自定义的 filter TokenAuthenticationFilter 外。

给 `authentication-manager` 取了一个别名 authenticationManager，是为了方便引用。

## 查看 session 是否创建

怎么知道什么时候创建了 session，什么时候没有创建呢？直接在 Tomcat 中看不够方便。

有个简单的办法是使用 Redis 保存 Tomcat 的 session，请参考 [Spring Security 集群](/spring-security-6-cluster)，只需要配置一下就可以，不需要写代码。

配置好后进入 **redis-cli**，先执行 **flushdb** 清空数据，然后使用 **auth-token** 的方式访问，redis-cli 中执行 **keys *** 看看有没有 session 生成，然后再使用浏览器访问，再执行 **keys *** 看看有没有 session 生成，反复的进行测试观察。

## 使用 auth-token 访问

使用 RESTful 的工具就可以了，例如 Firefox 的 RestClient 插件，Chrome 的 Postman 插件，或者自己使用 HttpClient 编程实现，请求时添加一个 header **auth-token** 就可以了。

## 问题与思考

### Token 怎么验证和存储？

上面只介绍了身份认证时使用 token，没有介绍怎么验证 token 是否有效，怎么获取 token 对应的用户信息。一种方案就是使用 Redis，token 为 key，用户信息为 value，并给 key 设置过期时间。

### Web 端访问时不使用 session 可以吗？

当然可以，但是 web 端如果不使用 session，就会失去一些 Spring Security 默认提供的特性，例如登陆成功后跳转到登陆前的页面这个功能就是使用 HttpSessionRequestCache 来存储 SavedRequest 实现的(UsernamePasswordAuthenticationFilter.successHandler)，还有如 RedirectAttributes 也需要 session，不使用 session 的时候，还需要这些功能的话，就得自己实现相关功能，把相关数据保存到 Redis，Cookie 等了。需要权衡有没有必要那么纯粹的不使用 session。