---
title: Spring + Fastweixin 微信开发
date: 2016-06-22 09:01:24
tags:
---

微信有两种模式，编辑模式和开发者模式，有些功能互斥的，不可以同时使用，微信开发需要在开发者模式下进行(开发者模式下仍然可以去微信的网页上群发消息)。下面介绍的功能能满足大部分的需求，响应文本消息，图文消息，创建菜单，响应菜单消息等。

我们给微信提供服务有两种消息模式，被动和主动

* 被动: 例如用户输入文本，点击菜单，微信服务器会访问我们的 Web 服务对应的 URL，我们返回对应的消息给微信服务器
* 主动: 例如创建菜单，群发消息，这种模式需要我们主动去触发，给微信服务器发送消息，可以是执行某个定时任务触发，或者我们访问某个 URL 然后在其响应的代码里触发

---

几个关键点

* 微信服务器和我们的服务器绑定验证时使用 GET 发送一个带有 `echostr` 参数的请求
* 其他消息使用的是 POST，常用的消息有
    1. 事件消息 event: subscribe, unsubscribe, location 等
    1. 文本消息 text: 可以回复文本，链接，图文
    2. 点击菜单回复的 click
    3. 点击菜单跳转为 view
* 微信服务器访问我们的服务器的 URL 只有一个，就是在配置页中配置的 URL
* 使用 app_id + app_secret 从微信服务器获取 access_token，有效时间是 7200 秒
* 微信的接口：用于我们主动的从微信服务器获取信息或者主动的向微信服务器写入信息

<!--more-->

## 准备公众号
1. 访问 <https://mp.weixin.qq.com> 注册开发者账号
2. 登陆后到最下面 `开发 > 基本配置` 中启用开发者模式，并点击 `修改配置` 配置我们自己给微信提供服务的 Web 地址、Token 等，然后使用 `appID` 和 `appsecret` 就可以进行开发了
    * 如果我们申请的是 `个人订阅号`，很多功能接口都没有，例如自定义菜单都没有，为了使用所有的功能进行开发练习，可以使用微信提供的 `公众平台测试帐号`：`开发 > 开发者工具 > 公众平台测试帐号 > 进入`，就可以看到 `appID` 和 `appsecret`，也需要在这里配置给微信提供服务的 Web 地址，Token 等，然后在页面中部找到 `请用微信扫描关注测试公众号`，扫描关注即可
    * [查看公众号接口权限说明](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1433401084&token=&lang=zh_CN): `开发 > 开发者工具 > 开发者文档 > 进入 > 公众号接口权限说明`
    * `个人订阅号` 是不能进行认证的

## Gradle 依赖
```groovy
compile("com.github.sd4324530:fastweixin:1.3.11")
```

> Fastweixin 的 git: <https://git.oschina.net/pyinjava/fastweixin>

## 给微信提供服务的 Controller
```java
public class WeixinController extends WeixinControllerSupport
```

## 消息响应
> 在 WeixinController 重载下面几个函数

* 响应订阅消息

    ```java
    protected BaseMsg handleSubscribe(BaseEvent event)
    ```
* 响应文本消息

    ```java
    protected BaseMsg handleTextMsg(TextReqMsg msg)
    ```

* 响应点击菜单消息

    ```java
    protected BaseMsg handleMenuClickEvent(MenuEvent event)
    ```

## 创建消息
* 创建文本消息

    ```java
    new TextMsg("你好: <a href=\"http://www.baidu.com\">百度</a>");
    ```

* 创建图文消息(单图文，多图文都可以)

    ```java
    String picUrl = "http://image.17car.com.cn/image/20120810/20120810092133_13289.jpg"; // 消息中显示的图片
    String url = "http://news.17car.com.cn/saishi/20120810/336283.html"; // 点击消息后跳转的网页的地址
    String description = "700 马力道路赛车 DDMWorks 打造最强 Atom";
    NewsMsg msg = new NewsMsg(Arrays.asList(new Article("Atom", description, picUrl, url), new Article("Atom", description, picUrl, url)));
    ```

## 创建菜单
> * 微信不会访问这个URL，需要我们自己访问创建菜单的 URL，然后才能向微信发送创建菜单信息。
> * 微信只能保证菜单 24 小时之类生效，想要马上看到菜单效果，先取消关注，然后再次关注就可以了
> * `CLICK` 类型的菜单会发送消息到我们的服务器，handleMenuClickEvent() 进行响应，根据 key 来判断是哪个菜单被点击
> * `VIEW` 类型的菜单不回发送消息到我们的服务器，而是直接跳转到对应的 URL
> * 菜单分一级菜单和二级菜单
>   * 最多有 3 个一级菜单，5 个二级菜单
>   * 一级菜单最多 4 个汉字，二级菜单最多 7 个汉字

```java
    @GetMapping("/create-menu")
    @ResponseBody
    public String createMenu() {
        // 准备一级主菜单
        MenuButton main1 = new MenuButton();
        main1.setType(MenuType.CLICK); // 可点击的菜单
        main1.setKey("main1");
        main1.setName("主菜单一");

        MenuButton main2 = new MenuButton();
        main2.setType(MenuType.VIEW); // 链接的菜单，点击后跳转到对应的 URL
        main2.setName("主菜单二");
        main2.setUrl("http://www.baidu.com");

        MenuButton main3 = new MenuButton();
        main3.setType(MenuType.CLICK);
        main3.setName("真题");

        // 带有子菜单
        MenuButton sub1 = new MenuButton();
        sub1.setType(MenuType.CLICK); // 带有子菜单
        sub1.setName("2016 语文");
        sub1.setKey("sub1");

        MenuButton sub2 = new MenuButton();
        sub2.setType(MenuType.CLICK);
        sub2.setName("2016 数学");
        sub2.setKey("sub2");
        main3.setSubButton(Arrays.asList(sub1, sub2));

        Menu menu = new Menu();
        menu.setButton(Arrays.asList(main1, main2, main3));

        //创建菜单
        ApiConfig config = new ApiConfig(APP_ID, APP_SECRET);
        MenuAPI menuAPI = new MenuAPI(config);
        ResultType resultType = menuAPI.createMenu(menu);
        return resultType.toString();
    }
```

## 使用 Json 字符串创建菜单
上面的程序创建菜单太麻烦了，可以使用 Json 字符串，然后反序列化为菜单对象，下面使用 Jackson 来实现。

> Jackson 依赖: `compile("com.fasterxml.jackson.core:jackson-databind:2.7.4")`

菜单的 Json 字符串可以放在文件，数据库中等，方便修改，而且比使用对象的方式更直观，例如下面这样:

```json
{
    "button": [{
        "type": "CLICK",
        "name": "主菜单一",
        "key": "main1"
    }, {
        "type": "VIEW",
        "name": "主菜单二",
        "url": "http://www.baidu.com"
    }, {
        "type": "CLICK",
        "name": "真题",
        "subButton": [{
            "type": "CLICK",
            "name": "2016 语文",
            "key": "sub1"
        }, {
            "type": "CLICK",
            "name": "2016 数学",
            "key": "sub2"
        }]
    }]
}
```

```java
    @GetMapping("/create-menu")
    @ResponseBody
    public String createMenu() throws Exception {
        String json = "{\n" +
                "    \"button\": [{\n" +
                "        \"type\": \"CLICK\",\n" +
                "        \"name\": \"今日歌曲\",\n" +
                "        \"key\": \"V1001_TODAY_MUSIC\"\n" +
                "    }, {\n" +
                "        \"name\": \"菜单\",\n" +
                "        \"subButton\": [{\n" +
                "            \"type\": \"VIEW\",\n" +
                "            \"name\": \"搜索\",\n" +
                "            \"url\": \"http://www.soso.com/\"\n" +
                "        }, {\n" +
                "            \"type\": \"VIEW\",\n" +
                "            \"name\": \"视频\",\n" +
                "            \"url\": \"http://v.qq.com/\"\n" +
                "        }, {\n" +
                "            \"type\": \"CLICK\",\n" +
                "            \"name\": \"赞一下我们\",\n" +
                "            \"key\": \"V1001_GOOD\"\n" +
                "        }]\n" +
                "    }]\n" +
                "}";

        ObjectMapper mapper = new ObjectMapper();
        Menu menu = mapper.readValue(json, Menu.class);
        
        //创建菜单
        ApiConfig config = new ApiConfig(APP_ID, APP_SECRET);
        MenuAPI menuAPI = new MenuAPI(config);
        ResultType resultType = menuAPI.createMenu(menu);
        return resultType.toString();
    }
```

## 示例程序
```java
package com.xtuer.controller;

import com.github.sd4324530.fastweixin.api.MenuAPI;
import com.github.sd4324530.fastweixin.api.config.ApiConfig;
import com.github.sd4324530.fastweixin.api.entity.Menu;
import com.github.sd4324530.fastweixin.api.entity.MenuButton;
import com.github.sd4324530.fastweixin.api.enums.MenuType;
import com.github.sd4324530.fastweixin.api.enums.ResultType;
import com.github.sd4324530.fastweixin.message.Article;
import com.github.sd4324530.fastweixin.message.BaseMsg;
import com.github.sd4324530.fastweixin.message.NewsMsg;
import com.github.sd4324530.fastweixin.message.TextMsg;
import com.github.sd4324530.fastweixin.message.req.MenuEvent;
import com.github.sd4324530.fastweixin.message.req.TextReqMsg;
import com.github.sd4324530.fastweixin.servlet.WeixinControllerSupport;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;

@RestController
@RequestMapping("/weixin")
public class WeixinController extends WeixinControllerSupport {
    private static final Logger LOG = LoggerFactory.getLogger(WeixinController.class);
    private static final String TOKEN      = "xxxxxx"; // 你的 token
    private static final String APP_ID     = "yyyyyy"; // 你的 appID
    private static final String APP_SECRET = "zzzzzz"; // 你的 appsecret

    // 重写父类方法，处理对应的微信消息
    @Override
    protected BaseMsg handleTextMsg(TextReqMsg msg) {
        String content = msg.getContent(); // content 是用户输入的信息

        switch (content.toUpperCase()) {
            case "URL":
                // 消息中有链接
                return new TextMsg("你好: <a href=\"http://www.baidu.com\">百度</a>");
            case "ATOM":
                // 图文消息
                String picUrl = "http://image.17car.com.cn/image/20120810/20120810092133_13289.jpg"; // 消息中显示的图片
                String url = "http://news.17car.com.cn/saishi/20120810/336283.html"; // 点击消息后跳转的网页的地址
                String description = "700 马力道路赛车 DDMWorks 打造最强 Atom";
                return new NewsMsg(Arrays.asList(new Article("Atom", description, picUrl, url), new Article("Atom", description, picUrl, url)));
            default:
                return new TextMsg("不识别的命令, 您输入的内容是: " + content);
        }
    }

    @Override
    protected BaseMsg handleMenuClickEvent(MenuEvent event) {
        String key = event.getEventKey();
        switch (key.toUpperCase()) {
            case "MAIN1":
                return new TextMsg("点击按钮");
            case "SUB1":
                return new TextMsg("2016 语文");
            case "SUB2":
                return new TextMsg("2016 数学");
            default:
                return new TextMsg("不识别的菜单命令");
        }
    }

    // 设置 TOKEN，用于绑定微信服务器
    @Override
    protected String getToken() {
        return TOKEN;
    }

    // 获取 access token: http://localhost:8080/weixin/access-token
    @GetMapping("/access-token")
    @ResponseBody
    public String getAccessToken() {
        ApiConfig config = new ApiConfig(APP_ID, APP_SECRET);
        return config.getAccessToken();
    }

    // 创建菜单, 访问  http://localhost:8080/weixincreate-menu 就会把菜单信息发送给微信服务器
    @GetMapping("/create-menu")
    @ResponseBody
    public String createMenu() {
        // 准备一级主菜单
        MenuButton main1 = new MenuButton();
        main1.setType(MenuType.CLICK); // 可点击的菜单
        main1.setKey("main1");
        main1.setName("主菜单一");

        MenuButton main2 = new MenuButton();
        main2.setType(MenuType.VIEW); // 链接的菜单，点击后跳转到对应的 URL
        main2.setName("主菜单二");
        main2.setUrl("http://www.baidu.com");

        MenuButton main3 = new MenuButton();
        main3.setType(MenuType.CLICK); // 带有子菜单
        main3.setName("真题");

        // 带有子菜单
        MenuButton sub1 = new MenuButton();
        sub1.setType(MenuType.CLICK);
        sub1.setName("2016 语文");
        sub1.setKey("sub1");

        MenuButton sub2 = new MenuButton();
        sub2.setType(MenuType.CLICK);
        sub2.setName("2016 数学");
        sub2.setKey("sub2");
        main3.setSubButton(Arrays.asList(sub1, sub2));

        Menu menu = new Menu();
        menu.setButton(Arrays.asList(main1, main2, main3));

        //创建菜单
        ApiConfig config = new ApiConfig(APP_ID, APP_SECRET);
        MenuAPI menuAPI = new MenuAPI(config);
        ResultType resultType = menuAPI.createMenu(menu);
        return resultType.toString();
    }
}
```
