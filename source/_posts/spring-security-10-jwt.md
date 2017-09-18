---
title: Spring Security JWT + Token 认证
date: 2017-09-03 08:08:21
tags: SpringSecurity
---

在 [Spring Security Session + Token 认证](http://qtdebug.com/spring-security-8-token/) 中介绍了 Token 相关的身份验证，但是怎么验证 token 和使用 token 获取用户信息没有进行介绍，可以把 token 存储到 Redis，下面介绍另一种 token 实现方法 JWT(Json Web Token)。JWT 中存储了 token 的签名，用户信息，还可以存储 token 的签发时间用于服务器验证 token 的有效期，并且这些信息如果被篡改了的话就会导致 token 失效，JWT 的理论请参考 <http://www.jianshu.com/p/576dbf44b2ae>。

为了在 Spring Security 中使用 JWT，需要修改下面 3 个类:

* TokenAuthenticationFilter
* TokenService
* JwtUtils <!--more-->

## TokenAuthenticationFilter

```java
package com.xtuer.security;

import com.xtuer.bean.User;
import org.springframework.beans.factory.annotation.Autowired;
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
    @Autowired
    private TokenService tokenService;

    private static ThreadLocal<Boolean> allowSessionCreation = new ThreadLocal<>(); // 是否允许当前请求创建 session

    public TokenAuthenticationFilter() {
        super(new AntPathRequestMatcher("/login", "POST")); // 参考 UsernamePasswordAuthenticationFilter
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException, IOException, ServletException {
        // 从 token 中提取 user，如果 user 不为 null，则用其创建一个 Authentication 对象
        String token = request.getHeader("auth-token");
        User user = tokenService.extractUser(token);

        if (user == null) {
            return null;
        } else {
            user.setPassword("no usage"); // 密码不能为 null，但是也没有用，所以随便设置一个吧
            user = User.userWithAuthorities(user); // 生成 authorities

            return new UsernamePasswordAuthenticationToken(user, user.getPassword(), user.getAuthorities());
        }
    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;

        allowSessionCreation.set(true); // 默认创建 session

        // 如果 header 里有 auth-token 时，则使用 token 查询用户数据进行登陆验证
        String token = request.getHeader("auth-token");

        if (token != null) {
            // 1. 尝试进行身份认证
            // 2. 如果用户无效，则返回 401
            // 3. 如果用户有效，则保存到 SecurityContext 中，供本次方式后续使用
            Authentication auth = attemptAuthentication(request, response);

            // user 不为 null 者身份验证成功
            if (auth != null) {
                // 保存认证信息到 SecurityContext，禁止 HttpSessionSecurityContextRepository 创建 session
                allowSessionCreation.set(false);
                SecurityContextHolder.getContext().setAuthentication(auth);
            } else  {
                response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Token 无效，请重新申请 token");
                return;
            }
        }

        // 继续调用下一个 filter: UsernamePasswordAuthenticationToken
        chain.doFilter(request, response);
    }

    public static boolean isAllowSessionCreation() {
        Boolean allow = allowSessionCreation.get();
        return allow == null ? true : allow; // 如果是 null，则说明没有设置过，使用默认的，也既是 true
    }
}
```

## TokenService

```java
package com.xtuer.security;

import com.alibaba.fastjson.JSON;
import com.xtuer.bean.User;
import com.xtuer.util.JwtUtils;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class TokenService {
    private String appSecret = "App secret"; // 应用的秘钥，可以定期更换
    private long tokenDuration = 3600L * 24 * 30 * 1000; // token 有效期为 30 天

    // 生成 token
    public String generateToken(User user) {
        return JwtUtils.generateToken(user, appSecret);
    }

    // 检测 token 的有效性
    public boolean checkToken(String token) {
        return JwtUtils.checkToken(token, appSecret, tokenDuration);
    }

    // 从 token 中提取用户数据
    public User extractUser(String token) {
        return JwtUtils.extractUser(token, appSecret, tokenDuration);
    }

    public static void main(String[] args) {
        TokenService service = new TokenService();

        // 创建用户对象
        User user = new User("Biao", "---", "ROLE_ADMIN", "ROLE_STAFF");
        user.setId(1234L);
        user.setMail("biao.mac@icloud.com");

        // 使用 user 生成 token
        String token = service.generateToken(user);
        System.out.println(token);

        // 检测 token 是否有效
        System.out.println(service.checkToken(token));

        // 从 token 中提取用户
        user = service.extractUser(token);
        System.out.println(JSON.toJSONString(user));
    }
}
```

## JwtUtils

```java
package com.xtuer.util;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.xtuer.bean.User;

import java.util.Map;
import java.util.TreeMap;

/**
 * 使用 JWT 的算法生成 token、验证 token 的有效性以及从 token 中提取数据。Token 中包含了用户数据、签名，并且能够防止 token 被篡改。
 * 增加或者删除不参与签名的数据已有 token 不会失效，增加或者删除参与签名的数据会使已有 token 失效.
 *
 * 标准 JWT 生成的 token 由 3 部分组成，这里对其进行了简化，去掉了算法说明的部分，保留了数据和签名部分.
 * 参考: http://www.jianshu.com/p/576dbf44b2ae
 *
 * 需要注意的是，放到 token 里的数据不要太多，否则会使得 token 很大，而 token 有可能放在 cookie, header 中，
 * 如果过大，容易被截断导致 token 无效.
 */
public class JwtUtils {
    /**
     * 使用 User 生成 token，由 2 部分组成，第一个部分为 payload 进行 Base64 编码的字符串，第二部分为签名.
     *
     * 算法:
     *     payload: 用户信息 + 签名生成时间进行 Base64 编码
     *     签名: 用户的关键信息(例如用户名，角色) + 签名生成时间 + 应用的秘钥使用 MD5 生成签名
     *
     * @param user 用户对象
     * @param secret 签名的秘钥匙
     * @return token 字符串
     */
    public static String generateToken(User user, String secret) {
        // 参与签名的数据: id, username, roles, signAt
        Map<String, String> params = new TreeMap<>(); // 使用 TreeMap 是为了 key 能够按照字母序进行排序
        params.put("id",       user.getId() + "");
        params.put("username", user.getUsername());
        params.put("roles",    JSON.toJSONString(user.getRoles()));
        params.put("signAt",   System.currentTimeMillis() + ""); // token 生成时间

        // 签名
        String signString = sign(params, secret);

        // 添加其他不参与签名的数据，例如邮件地址
        params.put("mail", user.getMail());
        String payload = JSON.toJSONString(params);

        // 使用 Url Safe Base64 进行编码是因为等号 = 会影响读取 cookie 中的值
        // 而 token 有可能放在 cookie, header, url 中
        return CommonUtils.base64UrlSafe(payload) + "." + signString;
    }

    /**
     * 检测 token 是否有效:
     *     1. token 的格式匹配 xxxxx.xxxxx
     *     2. token 的 signAt 需要在有效期内，否则无效
     *     3. 根据签名算法进行签名，计算出的签名和 token 中的签名相等则签名有效
     * @param token JWT token 字符串
     * @param secret 签名的秘钥匙
     * @param duration token 的有效期
     * @return token 有效时返回 true，无效时返回 false
     */
    public static boolean checkToken(String token, String secret, long duration) {
        // token 的格式: xxxxx.xxxxx，包含 0-9, a-z, A-Z, %
        if (token == null || !token.matches("[\\w%]+\\.[\\w]+")) {
            return false;
        }

        try {
            // 从 token 中得到用户信息的字符串和签名字符串
            int pos = token.indexOf(".");
            String payload = CommonUtils.unbase64UrlSafe(token.substring(0, pos));
            String signString = token.substring(pos+1);
            JSONObject json = JSON.parseObject(payload);

            // 检查签名的有效期: (currentTime - signAt) > DURATION 时无效
            long signAt = json.getLongValue("signAt");
            long elapsed = System.currentTimeMillis() - signAt;
            if (elapsed > duration) {
                return false;
            }

            // 使用 token 中的 id, username, roles, signAt 计算签名
            Map<String, String> params = new TreeMap<>();
            params.put("id",       json.getString("id"));
            params.put("username", json.getString("username"));
            params.put("roles",    json.getString("roles"));
            params.put("signAt",   json.getString("signAt"));

            String calculatedSignString = sign(params, secret);

            // 如果相等则签名没问题，不相等则签名被篡改，token 无效
            return signString.equals(calculatedSignString);
        } catch (Exception ex) {
            ex.printStackTrace(); // JSON 转换可能出错
        }

        return false;
    }

    /**
     * 从 token 中提取用户信息，如果 token 无效则返回 null.
     *
     * @param token JWT 生成的 token
     * @param secret 签名的秘钥匙
     * @param duration token 的有效期
     * @return 返回 token 中的用户对象，如果 token 无效则返回 null
     */
    public static User extractUser(String token, String secret, long duration) {
        if (!checkToken(token, secret, duration)) {
            return null;
        }

        try {
            int pos = token.indexOf(".");
            String payload = CommonUtils.unbase64UrlSafe(token.substring(0, pos));
            JSONObject json = JSON.parseObject(payload);
            JSONArray roles = JSON.parseArray(json.getString("roles"));
            json.put("roles", roles); // 注意: json 中的 roles 是字符串，需要转换为数组

            return JSON.parseObject(json.toJSONString(), User.class);
        } catch (Exception ex) {
            System.out.println(ex.getMessage());
        }

        return null;
    }

    /**
     * 签名计算
     */
    public static String sign(Map<String, String> params, String secret) {
        return CommonUtils.md5(JSON.toJSONString(params) + secret);
    }
}
```

