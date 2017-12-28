---
title: Spring Security JWT + Token 认证
date: 2017-09-03 08:08:21
tags: SpringSecurity
---

在 [Spring Security Session + Token 认证](http://qtdebug.com/spring-security-8-token/) 中介绍了 Token 相关的身份验证，但是怎么验证 token 和使用 token 获取用户信息没有进行介绍，可以把 token 存储到 Redis、数据库等，下面介绍另一种 token 实现方法 JWT(Json Web Token)，这种 token 不需要存储到服务器，自身就能进行验证。

JWT 中存储了 token 的签名，用户信息，还可以存储 token 的签发时间用于服务器验证 token 的有效期，并且这些信息如果被篡改了的话就会导致 token 失效，JWT 的理论请参考 <http://www.jianshu.com/p/576dbf44b2ae>。

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
import com.alibaba.fastjson.TypeReference;
import com.xtuer.bean.User;
import com.xtuer.util.Jwt;
import lombok.Getter;
import lombok.Setter;

import java.util.Map;

/**
 * 生成 token 的 service.
 */
@Getter
@Setter
public class TokenService {
    private String appId  = "School-1";
    private String appKey = "App secret"; // 应用的秘钥，可以定期更换
    private long tokenDuration = 3600L * 24 * 30 * 1000; // token 有效期为 30 天，单位为毫秒

    // 生成 token
    public String generateToken(User user) {
        // Token 中保存 id, username, roles
        long expiredAt = System.currentTimeMillis() + tokenDuration;
        return Jwt.create(appId, appKey).expiredAt(expiredAt)
                .param("id", user.getId() + "")
                .param("username", user.getUsername())
                .param("roles", JSON.toJSONString(user.getRoles().toArray(new String[0])))
                .token();
    }

    // 检测 token 的有效性
    public boolean checkToken(String token) {
        return Jwt.checkToken(token, appKey);
    }

    // 从 token 中提取用户
    public User extractUser(String token) {
        if (!this.checkToken(token)) {
            return null;
        }

        try {
            // 获取 token 中保存的 id, username, roles
            Map<String, String> params = Jwt.params(token);
            Long         id = Long.parseLong(params.get("id"));
            String username = params.get("username");
            String[]  roles = JSON.parseObject(params.get("roles"), new TypeReference<String[]>() {});

            return new User(id, username, "no-password", roles);
        } catch (Exception ex) {
            return null;
        }
    }

    public static void main(String[] args) {
        TokenService service = new TokenService();

        // 创建用户对象
        User user = new User(1234L, "Biao", "---", "ROLE_ADMIN", "ROLE_STAFF");
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

## Jwt

```java
package com.xtuer.util;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONException;
import com.alibaba.fastjson.TypeReference;
import lombok.Getter;
import lombok.Setter;
import org.apache.commons.lang3.StringUtils;
import org.springframework.util.Assert;

import java.util.Collections;
import java.util.Map;
import java.util.TreeMap;

/**
 * 使用 JWT 的算法生成 token、验证 token 的有效性以及从 token 中提取数据。Token 中可包含用户数据、签名，能够防止 token 被篡改。
 *
 * 标准 JWT 生成的 token 由 3 部分组成，这里对其进行了简化，去掉了算法说明的部分，保留了数据和签名部分.
 * 参考: http://www.jianshu.com/p/576dbf44b2ae
 *
 * 需要注意的是，放到 token 里的数据不要太多，否则会使得 token 很大，而 token 有可能放在 cookie, header 中，
 * 如果过大，容易被截断导致 token 无效.
 *
 * 为什么不使用 com.auth0:java-jwt:3.3.0 实现的 JWT 呢？因为他的算法在 Nginx 端实现是不够方便。
 *
 * 使用方法:
 * 生成 token: Jwt.create(appId, appKey).param("username", "放下").expiredAt(System.currentTimeMillis() + 2000).token()
 * 校验 token: Jwt.checkToken(token, appKey)
 * 提取 token 中的用户数据: Jwt.params(token)
 */
public class Jwt {
    /**
     * 检查 token 是否有效: 使用 payload 计算的签名结果和 token 中的签名一样，如果还存在有效期 expiredAt 并且未过期，则签名有效.
     *
     * @param jwtToken JWT token
     * @param appKey   应用的 Key
     * @return 签名有效返回 true，否则返回 false.
     */
    public static boolean checkToken(String jwtToken, String appKey) {
        if (StringUtils.isBlank(jwtToken)) {
            return false;
        }

        int dotIndex = jwtToken.indexOf(".");
        if (dotIndex == -1) {
            return false;
        }

        try {
            // 1. 解析出参数的 map params
            // 2. 如果 params 中存在 expiredAt，如果 expiredAt 超过当前时间则 token 过期无效
            // 3. 如果 token 未过期，则用 appKey+params 计算签名，如果和 signature 相等则签名有效
            Map<String, String> params = Jwt.params(jwtToken);

            try {
                // 检查签名是否过期
                Long expiredAt = Long.parseLong(params.get("expiredAt"));
                if (expiredAt < System.currentTimeMillis()) {
                    return false;
                }
            } catch (NumberFormatException ex) {}

            String signature = jwtToken.substring(dotIndex+1);
            return signature.equals(Jwt.sign(params, appKey));
        } catch (JSONException ex) {
            return false;
        }
    }

    /**
     * 获取 token 中的 payload 的 map.
     *
     * @param jwtToken JWT token
     * @return 返回 payload 的 map，如果 token 无效则返回空的 map.
     */
    public static Map<String, String> params(String jwtToken) {
        int dotIndex = jwtToken.indexOf(".");
        if (dotIndex == -1) {
            return Collections.emptyMap();
        }

        String payload = Utils.unbase64UrlSafe(jwtToken.substring(0, dotIndex));

        try {
            return JSON.parseObject(payload, new TypeReference<TreeMap<String, String>>() {});
        } catch (NumberFormatException ex) {
            return Collections.emptyMap();
        }
    }

    private static String sign(Map<String, String> params, String appKey) {
        // 初始化用来计算签名的字符串 toSignedText 为 appKey，
        // 然后按照 params 中 key 的字母序遍历 params，value 挨个的加在到 toSignedText 后面
        Map<String, String> sortedMap = new TreeMap<>(params);
        StringBuilder toSignedText = new StringBuilder(appKey);

        for (Map.Entry<String, String> entry : sortedMap.entrySet()) {
            toSignedText.append(entry.getValue());
        }

        return Utils.md5(toSignedText.toString());
    }

    /**
     * 使用 appId 和 appKey 创建一个 JWT 的 builder，然后使用此 builder 设置 payload 的参数计算 token.
     *
     * @param appId  应用的 ID
     * @param appKey 应用的 key
     * @return 返回 builder 对象
     */
    public static Builder create(String appId, String appKey) {
        return new Builder(appId, appKey);
    }

    @Getter
    @Setter
    public static class Builder {
        private Long expiredAt; // token 过期时间
        private String appId;   // 应用的 ID
        private String appKey;  // 应用的 key
        private TreeMap<String, String> params = new TreeMap<>(); // payload 的参数

        public Builder(String appId, String appKey) {
            Assert.notNull(appId, "JWT appId cannot be null");
            Assert.notNull(appKey, "JWT appKey cannot be null");
            this.appId = appId;
            this.appKey = appKey;
        }

        /**
         * 设置 token 的过期时间
         *
         * @param expiredAt 过期时间，单位是毫秒
         * @return 返回 builder 自己
         */
        public Builder expiredAt(long expiredAt) {
            this.expiredAt = expiredAt;
            return this;
        }

        /**
         * 添加用户数据到 token 中
         *
         * @param name  数据的 key
         * @param value 数据的 value
         * @return 返回 builder 自己
         */
        public Builder param(String name, String value) {
            Assert.notNull(name, "JWT param name cannot be null");
            Assert.notNull(value, "JWT param value cannot be null");
            params.put(name, value);
            return this;
        }

        /**
         * 使用 appId, appKey, signedAt [, expiredAt], params 生成 token.
         * 生成算法为:
         *     1. 添加 appId, signedAt[, 如果 expiredAt 不为 null 也加入] 到 params 中
         *     2. 初始化用来计算签名的字符串 toSignedText 为 appKey
         *     3. 按照 params 中 key 的字母序遍历 params，value 挨个的加在到 toSignedText 后面
         *     4. signature = MD5(toSignedText)
         *     5. 把 params 转换为 JSON 字符串并使用 URL Save 的 BASE64 对其进行编码得到 payload
         *     6. 最后得到的签名结果为 payload.signature
         *
         * @return 返回使用 JWT 签名的字符串
         */
        public String token() {
            // 添加签名需要的数据项
            params.put("appId", appId);
            params.put("signedAt", System.currentTimeMillis()+"");
            if (expiredAt != null) {
                params.put("expiredAt", expiredAt+"");
            }

            // 计算签名
            String payload   = Utils.base64UrlSafe(JSON.toJSONString(params));
            String signature = Jwt.sign(params, appKey);
            return payload + "." + signature;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        String appId  = "school-1";
        String appKey = "Passw0rd";
        String token;

        // 1. 没有期限的 token, 一直有效
        token = Jwt.create(appId, appKey).param("username", "放下").token();
        System.out.println(token);
        System.out.println(Jwt.checkToken(token, appKey));
        System.out.println(StringUtils.repeat("-", 120));

        // 2. 有效期为 2 秒，2 秒后过期
        token = Jwt.create(appId, appKey).param("username", "放下").expiredAt(System.currentTimeMillis() + 2000).token();
        System.out.println(token);
        System.out.println(Jwt.checkToken(token, appKey));
        Thread.sleep(2500);
        System.out.println(Jwt.checkToken(token, appKey));

        System.out.println(Jwt.params(token));
        System.out.println(StringUtils.repeat("-", 80));

        // 3. 乱给的 token, 无效
        System.out.println(Jwt.checkToken("", appKey));
        System.out.println(Jwt.checkToken("xxx", appKey));
        System.out.println(Jwt.checkToken("xxx.yyy", appKey));
    }
}
```

