---
title: OkHttp
date: 2016-11-27 18:14:43
tags: [Java, Util]
---

> 在 Java 平台上，我们一般使用 Apache HttpClient 作为通常的 HTTP 客户端。Square 公司开源的 OkHttp 是一个更先进的专注于连接效率的 HTTP 客户端。OkHttp 提供了对 HTTP/2 和 SPDY 的支持，并提供了连接池，GZIP 压缩和 HTTP 响应缓存功能。OkHttp 的 API 接口也更加的简单实用。可以将 OkHttp 作为 Apache HttpClient 的升级与替换，本文将对其进行详细的介绍，更详细内容请参考 <https://www.ibm.com/developerworks/cn/java/j-lo-okhttp>

<!--more-->

```java
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

import java.io.IOException;

public class OkHttpTest {
    public static void main(String[] args) throws IOException {
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder().url("http://www.baidu.com").build();
        Response response = client.newCall(request).execute();

        if (!response.isSuccessful()) {
            System.out.println("Error: " + response);
            return;
        }

        System.out.println(response.body().string());
    }
}
```

## Gradle 依赖

```groovy
compile 'com.squareup.okhttp3:okhttp:3.4.2'
```

