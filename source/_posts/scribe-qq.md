---
title: QQ 登陆的 ScribeJava 实现
date: 2016-10-04 15:23:07
tags: Java
---
QQ 提供了标准的 OAuth 2.0 第三方登录功能，可以按照 QQ 互联的文档 (<http://wiki.connect.qq.com>) 例如使用 HttpClient 进行网络访问，集成 QQ 的第三方登录到我们的系统里，也可以使用 `Scribe-Java` 来集成 QQ 的第三方登录，这样就会简单很多。

ScribeJava 是一个简单的 Java 实现的 OAuth/OAuth2 库。  
ScribeJava Github 仓库：<https://github.com/scribejava/scribejava>

<!--more-->

## 注册 QQ 互联账号
1. 在开发前，需要在 `QQ 互联` 注册一个开发者账号: <https://connect.qq.com>
2. 然后点击 `应用管理`: <https://connect.qq.com/manage.html>
3. 创建 `网站应用`，里面有开发需要的 `APP ID` 和 `APP Key`

## 修改 hosts
例如我们在 QQ 互联中填写的回调 URL 为 <http://open.qtdebug.com:8080/oauth/qq/callback>，很显然 QQ 服务器是不能访问这个地址的，因为这是我们本地的地址，只有我们机器上能访问，为了在 QQ 登陆成功后 QQ 服务器能访问这个地址，需要在系统的 `hosts` 文件里添加 `127.0.0.1 open.qtdebug.com`。

还有另一种方式是使用如 `Ngrok` 把本地映射为外网可访问。

## Gradle 依赖
为了使用 ScribeJava，需要依赖

```groovy
compile 'com.github.scribejava:scribejava-apis:3.2.0'
```

为了使用 FastJson 解析 QQ 返回的 JSON 响应，需要依赖

```groovy
compile 'com.alibaba:fastjson:1.2.17'
```

## QQApi
`ScribeJava` 已经集成了了几十个网站登陆的 Service，不幸的是没有提供 QQ 的，不过没关系，我们可以使用 `ada.young` 实现的 `QQApi`，在 <http://git.oschina.net/cng1985/scribejava> 上下载，把 `QQApi.java` 和 `OsChinaOAuthServiceImpl.java` 放到我们的工程中即可。

## OAuth 的流程
![](/img/spring/oauth20-flow.png)

## QQ 登陆的代码
先放上 QQ 登陆的代码 `QQOAuthController.java` 和 `QQOAuthService.java`，然后再对其进行讲解。

`QQOAuthController.java`

```java
package com.xtuer.controller;

import com.alibaba.fastjson.JSONObject;
import com.xtuer.service.QQOAuthService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Controller
public class QQOAuthController {
    @Autowired
    private QQOAuthService qqOAuthService;

    // 访问登陆页面，然后会重定向到 QQ 的登陆页面
    @GetMapping("/oauth/qq")
    public String qqLogin() {
        return "redirect:" + qqOAuthService.getLoginUrl();
    }

    // QQ 成功登陆后的回调
    @GetMapping("/oauth/qq/callback")
    @ResponseBody
    public String qqLoginCallback(@RequestParam("code") String code, HttpServletResponse response) throws IOException {
        String accessToken = qqOAuthService.getAccessToken(code); // 5943BF2461ED97237B878BECE78A8744

        // 保存 accessToken 到 cookie，过期时间为 30 天，便于以后使用
        Cookie cookie = new Cookie("accessToken", accessToken);
        cookie.setMaxAge(60 * 24 * 30);
        response.addCookie(cookie);

        return accessToken;
    }

    // 获取 QQ 用户的信息
    @GetMapping("/oauth/qq/user")
    @ResponseBody
    public String getUserInfo(@CookieValue(name = "accessToken", required = false) String accessToken) throws IOException {
        if (accessToken == null) {
            return "There is no access token, please login first!";
        }

        String openId = qqOAuthService.getOpenId(accessToken);
        JSONObject json = qqOAuthService.getUserInfo(accessToken, openId);

        return json.toJSONString();
    }
}
```

`QQOAuthService.java`

```java
package com.xtuer.service;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.github.scribejava.apis.QQApi;
import com.github.scribejava.core.builder.ServiceBuilder;
import com.github.scribejava.core.model.OAuth2AccessToken;
import com.github.scribejava.core.model.OAuthRequest;
import com.github.scribejava.core.model.Response;
import com.github.scribejava.core.oauth.OAuth20Service;
import org.springframework.stereotype.Service;

import java.io.IOException;

@Service
public class QQOAuthService {
    // 获取用户 openid 的 URL
    private static final String OPEN_ID_URL = "https://graph.qq.com/oauth2.0/me";

    // 获取用户信息的 URL，oauth_consumer_key 为 apiKey
    private static final String USER_INFO_URL = "https://graph.qq.com/user/get_user_info?oauth_consumer_key=%s&openid=%s";

    // 下面的属性可以通过配置读取
    private String callbackUrl = "http://open.qtdebug.com:8080/oauth/qq/callback"; // QQ 在登陆成功后回调的 URL，这个 URL 必须在 QQ 互联里填写过
    private String apiKey      = "101292272";                                      // QQ 互联应用管理中心的 APP ID
    private String apiSecret   = "5bdbe9403fcc3abe8eba172337904b5a";               // QQ 互联应用管理中心的 APP Key
    private String scope       = "get_user_info";                                  // QQ 互联的 API 接口，访问用户资料

    private OAuth20Service oauthService; // 访问 QQ 服务的 service

    public QQOAuthService() {
        // 创建访问 QQ 服务的 service
        oauthService = new ServiceBuilder().apiKey(apiKey).apiSecret(apiSecret)
                .scope(scope).callback(callbackUrl).build(QQApi.instance());
    }

    /**
     * 取得 QQ 登陆页面的 URL，例如
     * https://graph.qq.com/oauth2.0/authorize?response_type=code&client_id=101292272&
     * redirect_uri=http://open.qtdebug.com:8080/oauth/qq/callback&scope=get_user_info
     *
     * @return QQ 登陆页面的 URL
     */
    public String getLoginUrl() {
        return oauthService.getAuthorizationUrl();
    }

    /**
     * 使用 code 换取 access token
     *
     * @param code 成功登陆后 QQ Server 返回给回调 URL 的中间 code，用于换取 access token
     * @return 用于访问 QQ 服务的 token
     * @throws IOException
     */
    public String getAccessToken(String code) throws IOException {
        OAuth2AccessToken token = oauthService.getAccessToken(code); // 使用 code 换取 accessToken
        String accessToken = token.getAccessToken(); // 5943BF2461ED97237B878BECE78A8744

        return accessToken;
    }

    /**
     * 获取用户的 open id，每个用户对于同一个 APP ID 的 open id 是一样的
     *
     * @param accessToken 登陆时从 QQ 系统得到的 access token，作为访问的凭证，相当于用户名密码的作用
     * @return 用户的 open id
     * @throws IOException
     */
    public String getOpenId(String accessToken) throws IOException {
        Response oauthResponse = request(oauthService, accessToken, OPEN_ID_URL);

        // 消息格式: callback( {"client_id":"101292272","openid":"4584E3AAABFC5F052971C278790E9FCF"} );
        String responseBody = oauthResponse.getBody();
        int s = responseBody.indexOf("{");
        int e = responseBody.lastIndexOf("}") + 1;
        String json = responseBody.substring(s, e);
        JSONObject obj = JSON.parseObject(json);

        return obj.getString("openid");
    }

    /**
     * 获取用户的信息，QQ 的昵称，QQ 空间的头像等，一般这 2 个属性用的最多
     *
     * @param accessToken 登陆时从 QQ 系统得到的 access token，作为访问的凭证，相当于用户名密码的作用
     * @param openId 用户的 open id
     * @return JSONObject 对象
     * @throws IOException
     */
    public JSONObject getUserInfo(String accessToken, String openId) throws IOException {
        String url = String.format(USER_INFO_URL, apiKey, openId);
        Response oauthResponse = request(oauthService, accessToken, url);
        String responseJson = oauthResponse.getBody();

        return JSON.parseObject(responseJson);
    }

    /**
     * 使用 OAuth 2.0 的方式从服务器获取 URL 指定的信息
     *
     * @param accessToken 登陆时从 QQ 系统得到的 access token，作为访问的凭证，相当于用户名密码的作用
     * @param url 访问 OAuth Server 服务的 URL
     * @return
     */
    public Response request(OAuth20Service service, String accessToken, String url) {
        OAuth2AccessToken token = new OAuth2AccessToken(accessToken);
        OAuthRequest oauthRequest = new OAuthRequest(service.getApi().getAccessTokenVerb(), url, service);
        service.signRequest(token, oauthRequest); // 会把 accessToken 添加到请求中，GET 请求即添加到 URL 上
        Response oauthResponse = oauthRequest.send();

        return oauthResponse;
    }
}
```

## 代码讲解
* 创建 OAuth20Service 对象，用于访问 QQ 服务

    ```java
    oauthService = new ServiceBuilder().apiKey(apiKey).apiSecret(apiSecret)
                .scope(scope).callback(callbackUrl).build(QQApi.instance());
    ```
* 登陆访问 <http://open.qtdebug.com:8080/oauth/qq>，自动重定向到 QQ 登陆页面，登陆 URL 调用 `oauthService.getAuthorizationUrl()` 获得。
* QQ 登陆成功后，回调到 `QQOAuthController.qqLoginCallback()`，调用 `qqOAuthService.getAccessToken(code)` 取得 `access token`，有效期为 3 个月。为了方便以后使用这个 access token，所以把它存储到 cookie。
* ScribeJava 只提供了登陆和获取 accessToken 的功能，例如想要获取用户的 open id 等其他信息就必须我们自己实现了，所以上面的代码封装了 `QQOAuthService.request()`，用于访问 OAuth Server，具体使用请参考 `QQOAuthService.getOpenId()` 和 `QQOAuthService. getUserInfo()`:
    * `QQOAuthService.getOpenId()` 返回的数据格式为

        ```json
        callback( {"client_id":"101292272","openid":"4584E3AAABFC5F052971C278790E9FCF"} );
        ```

    * `QQOAuthService. getUserInfo()` 返回的数据格式为

        ```json
        {
            "ret": 0,
            "msg": "",
            "is_lost":0,
            "nickname": "公孙二狗",
            "gender": "男",
            "province": "北京",
            "city": "东城",
            "year": "2013",
            "figureurl": "http:\/\/qzapp.qlogo.cn\/qzapp\/101292272\/4584E3AAABFC5F052971C278790E9FCF\/30",
            "figureurl_1": "http:\/\/qzapp.qlogo.cn\/qzapp\/101292272\/4584E3AAABFC5F052971C278790E9FCF\/50",
            "figureurl_2": "http:\/\/qzapp.qlogo.cn\/qzapp\/101292272\/4584E3AAABFC5F052971C278790E9FCF\/100",
            "figureurl_qq_1": "http:\/\/q.qlogo.cn\/qqapp\/101292272\/4584E3AAABFC5F052971C278790E9FCF\/40",
            "figureurl_qq_2": "http:\/\/q.qlogo.cn\/qqapp\/101292272\/4584E3AAABFC5F052971C278790E9FCF\/100",
            "is_yellow_vip": "0",
            "vip": "0",
            "yellow_vip_level": "0",
            "level": "0",
            "is_yellow_year_vip": "0"
        }
        ```

## 思考
拿到用户的 open id 和昵称等后怎么用呢？

一般都会在用户登陆成功后回调 `QQOAuthController.qqLoginCallback()` 中得到 access token，然后取得用户的 open id，用它在我们的系统中查看此 open id 是否有对应的账号，如果有则进行自动登录，如果没有，则进入账号绑定页面，询问是创建新账号还是绑定已有账号，其中又有可能需要获取用户的昵称，大致逻辑如下:

```java
// QQ 成功登陆后的回调
@GetMapping("/oauth/qq/callback")
public String qqLoginCallback(@RequestParam("code") String code, HttpServletResponse response) throws IOException {
    String accessToken = qqOAuthService.getAccessToken(code); // 5943BF2461ED97237B878BECE78A8744

    // 保存 accessToken 到 cookie，过期时间为 30 天，便于以后使用
    Cookie cookie = new Cookie("accessToken", accessToken);
    cookie.setMaxAge(60 * 24 * 30);
    response.addCookie(cookie);

    // TODO
    String openId = qqOAuthService.getOpenId(accessToken);
    User user = UserDao.findUserByOpenId("QQ_" + openId); // 加上前缀避免冲突

    if (user == null) {
        // 1. Navigate to binding account page
        // 2. Create new account or binding with existing account
        return "redirect:/bindingAccountPage.html";
    } else {
        // 进行自动登录: 用 openId 查找本地的对应账号，用此账号进行自动登陆
    }

    return "登陆结果.html";
}
```

> 第三方账号和本地的用户账号一般也会使用一个中间表进行关联，例如  
> 
> id | use_id | third_party_account_id | type
> -- | ------ | -------- | ----
> 1  | 100000 | QQ_AS23DCF3 | QQ
> 2  | 100001 | WX_BDAD1D3F | Weixin

