---
title: Nginx 验证 Token
date: 2017-10-15 17:12:06
tags: Mac
---

为了提高效率，常把 Nginx 作为静态文件服务器，把视频文件，JS，CSS 等放到 Nginx 上。例如我们要开发一个视频网站，免费视频不需要访问权限验证，收费视频就需要对用户的权限进行验证，验证通过了才能够继续访问，Nginx 可以借助 Lua 来实现访问验证，用户信息使用 token 表示

* 计算 token: md5(appId+appKey)
* 请求的链接从应用服务器上动态获取，请求参数带上 appId 和 token: http://localhost/private/tih.mp4?appId=app_1&token=1409dc951714a8226032e0b0fb60bdb0
* Nginx 接收到请求的时候，根据 appId 找到 appKey，然后 token2 = ngx.md5(appId+appKey)，token2 和请求中的 token 比较，如果相等则验证通过放行访问，否则禁止访问

Nginx 简单的验证代码如下:

```js
location ~ /private/.+\.mp4$ {
    root html;

    access_by_lua '
        -- 应用的 ID 和 key，和应用服务器上的一致
        local appIdKeys = {["app_1"] = "key_1", ["app_2"] = "key_2"};

        local args   = ngx.req.get_uri_args();
        local appId  = args["appId"];
        local appKey = appIdKeys[appId];

        local token1 = args["token"]; -- 参数中 token
        local token2 = ngx.md5(appId .. appKey); -- 用应用的 ID 找到对应的 key，然后根据算法计算 token

        -- 如果参数中的 token 和计算得到的 token 不相等，则说明访问非法，禁止访问，否则放行访问
        if token1 ~= token2 then
            ngx.exit(ngx.HTTP_FORBIDDEN);
        end
    ';
}
```

Nginx 和应用服务器上同时存储 appId 和 appKey，这样就能根据参数中的 appId 查找到对应的 appKey。至于使用 Lua 的变量存储，或者使用数据库，还是文件，根据具体的情况而定(Nginx 中 Lua 能够访问数据、Redis 等)。

上面的验证规则比较简单，如果其他人得到了 token，就可以无限制的访问了，为了增强安全性，可以使用更多的参数生成 token，例如用户 id，限制 URL 期限的时间戳等。

> Nginx 默认没有安装 Lua 模块，需要自己安装，可参考 <http://qtdebug.com/mac-nginx-lua>