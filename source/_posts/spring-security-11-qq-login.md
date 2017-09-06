---
title: Spring Security QQ 登陆
date: 2017-09-05 19:23:37
tags: Spring-Security
---

Spring Security 中实现 QQ 登陆，可以在 FORM_LOGIN_FILTER 前插入一个 filter 用于拦截 QQ 登陆成功后的回调，进行身份认证。

开发前需要准备一个 QQ 互联账号和修改 hosts，按照下面的说明操作即可。

> 要点: Spring Security 中身份认证成功的标志很简单，只要用用户信息创建一个 Authentication 对象，保存到 SecurityContextHolder 就可以了。
>
> Spring Security 发现 SecurityContextHolder 中有 Authentication 后，就认为用户已经通过了身份认证，对访问的资源进行权限验证时调用 Authentication.getAuthorities() 获取用户的权限进行验证。

## 注册 QQ 互联账号

1. 在开发前，需要在 `QQ 互联` 注册一个开发者账号: [https://connect.qq.com](https://connect.qq.com/)
2. 然后点击 `应用管理`: <https://connect.qq.com/manage.html>
3. 创建 `网站应用`，里面有开发需要的 `APP ID` 和 `APP Key`

## 修改 hosts

例如我们在 QQ 互联中填写的回调 URL 为 <http://open.qtdebug.com:8080/oauth/qq/callback>，很显然 QQ 服务器是不能访问这个地址的，因为这个地址不存在，为了在 QQ 登陆成功后 QQ 服务器能访问这个地址，需要在系统的 `hosts` 文件里添加 `127.0.0.1 open.qtdebug.com`。

还有另一种方式是使用如 `Ngrok` 把本地映射为外网可访问。<!--more-->

## Gradle 依赖

使用 EasyOkHttp 访问网络

```groovy
compile 'com.mzlion:easy-okhttp:1.1.3'
```

为了使用 FastJson 解析 QQ 返回的 JSON 响应，需要依赖

```groovy
compile 'com.alibaba:fastjson:1.2.17'
```

## 登陆按钮

在登陆页面放置一个登陆连接，点击后跳到 QQ 登陆页面

```html
<a href="https://graph.qq.com/oauth2.0/authorize?response_type=code&client_id=101292272&redirect_uri=http://open.qtdebug.com:8080/oauth/qq/callback&scope=get_user_info">QQ Login</a>
```

## OAuthAuthenticationFilter

当 doFilter() 中发现请求的 URI 为 /oauth/qq/callback 时，则说明是 QQ 登陆成功的回调地址，接下来就是根据 [QQ 登陆的 API](http://wiki.connect.qq.com/使用authorization_code获取access_token) 一步一步的请求用户的数据，直到拿到用户的 open id，使用 open id 去查询系统中是否有账号与之对应，有的话把用户信息保存到 SecurityContextHolder，即身份认证成功，跳转到登陆前的页面，如果此 open id 不存在与之对应的用户，则跳转到用户绑定页面引导用户创建或者和已有账户绑定，把此账户信息保存到 SecurityContextHolder，然后跳转到登陆前的页面。

>  注意: QQ 登陆后不能继续执行下一个 filter。

```java
package com.xtuer.security;

import com.alibaba.fastjson.JSON;
import com.mzlion.easyokhttp.HttpClient;
import com.xtuer.bean.User;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.DefaultRedirectStrategy;
import org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class OAuthAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    // 替换为自己的 client id 和 client secret
    private String qqClientId = "101292272";
    private String qqClientSecret = "5bdbe9403fcc3abe8eba172337904b5a";

    private String QQ_ACCESS_TOKEN_URL = "https://graph.qq.com/oauth2.0/token?grant_type=authorization_code&client_id=%s&client_secret=%s&redirect_uri=%s&code=%s";
    private String QQ_OPEN_ID_URL = "https://graph.qq.com/oauth2.0/me?access_token=%s";
    private String QQ_CALLBACK = "http://open.qtdebug.com:8080/oauth/qq/callback";

    public OAuthAuthenticationFilter() {
        super("/");
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException, IOException, ServletException {
        return null;
    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;

        // 被拦截到说明是 QQ 登陆成功的回调地址 http://host:port/oauth/qq/callback
        if (request.getRequestURI().startsWith("/oauth/qq/callback")) {
            // [1] 获取 code
            String code = request.getParameter("code");
            System.out.println("Code: " + code);

            // [2] 用 code 换取 access token
            // 响应: access_token=1A2CF189A4BBEE25CACE587CDD106512&expires_in=7776000&refresh_token=A5A3B6D90955ED6934EC42F2EECDA4BC
            String accessTokenUrl = String.format(QQ_ACCESS_TOKEN_URL, qqClientId, qqClientSecret, QQ_CALLBACK, code);
            String responseData = HttpClient.get(accessTokenUrl).execute().asString();
            String token = responseData.replaceAll("access_token=(.+)&expires_in=.+", "$1");
            System.out.println("Access Token: " + token);

            // [3] 用 access token 获取用户的 open ID
            // 响应: callback( {"client_id":"101292272","openid":"4584E3AAABFC5F052971C278790E9FCF"} );
            String openIdUrl = String.format(QQ_OPEN_ID_URL, token);
            responseData =HttpClient.get(openIdUrl).execute().asString();
            int start = responseData.indexOf("{");
            int end = responseData.lastIndexOf("}") + 1;
            String json = responseData.substring(start, end);
            String openId = JSON.parseObject(json).getString("openid");
            System.out.println("Open ID: " + openId);

            // [4] 使用 openId 查找用户
            User user = new User("admin", "----", "ROLE_ADMIN"); // 假设 admin 是使用 open id 查找到的用户吧
            // user = null; // user 赋值为 null，表示没找到用户

            if (user != null) {
                // [5] 用户存在，登陆成功，跳转到登陆前的页面
                Authentication auth = new UsernamePasswordAuthenticationToken(user, user.getPassword(), user.getAuthorities());
                super.successfulAuthentication(request, response, chain, auth); // 跳转到登陆前页面
            } else {
                // [6] 用户不存在，跳转到 "创建|绑定已有用户" 页面，
                // 绑定好用户后保存用户信息到: SecurityContextHolder.getContext().setAuthentication(auth)
                // 然后跳转到登陆前的页面
                DefaultRedirectStrategy redirectStrategy = new DefaultRedirectStrategy();
                redirectStrategy.sendRedirect(request, response, "/page/bindUser");
            }

            return;
        } else if (request.getRequestURI().startsWith("/oauth/weixin/callback")) {

        }

        chain.doFilter(request, response);
    }
}
```

### 绑定用户的逻辑:

1. OAuthAuthenticationFilter 中重定向到 **/page/bindUser**
2. 用户填写账号相关信息
3. 提交表单到 **/form/bindUsers**
4. 处理用户信息，身份认证
5. 跳转到登陆前的页面

```java
@PostMapping("/form/bindUsers")
public String bindUser(HttpServletRequest request, HttpServletResponse response) {
    // 1. 绑定已有用户或者创建用户
    User user = new User(...);
    
    // 2. 保存用户信息到 SecurityContextHolder，身份认证成功
    Authentication auth = new UsernamePasswordAuthenticationToken(user, user.getPassword(), user.getAuthorities());
    SecurityContextHolder.getContext().setAuthentication(auth);

    // 3. 重定向到登陆前的页面
    SavedRequest savedRequest = new HttpSessionRequestCache().getRequest(request, response);
    String redirectUrl = (savedRequest != null) ? savedRequest.getRedirectUrl() : "/";

    return redirectUrl;
}
```

## spring-security.xml

```xml
<http auto-config="true">
    <intercept-url pattern="/page/admin"   access="ROLE_ADMIN"/>
    <intercept-url pattern="/demo/filters" access="ROLE_USER"/>
    ...
    <custom-filter ref="oauthAuthenticationFilter" before="FORM_LOGIN_FILTER"/>
</http>

<!-- 第三方登陆 filter -->
<beans:bean id="oauthAuthenticationFilter" class="com.xtuer.security.OAuthAuthenticationFilter">
    <beans:property name="authenticationManager" ref="authenticationManager"/>
</beans:bean>
```

## 优化

上面 QQ 的 OAuth 认证部分代码优点是简单、直接，但是太粗暴、丑陋了些，为了更好地组织代码，可以使用 OAuth 的框架 ScribeJava  进行重构，参考 [QQ 登陆的 Scribe-Java 实现](http://qtdebug.com/scribe-qq/)。

## 思考

我们只提供了 QQ 登陆的实现，想要微信扫码登陆、微信公众号登陆、微博登陆时应该怎么做呢？