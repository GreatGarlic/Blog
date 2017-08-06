---
title: OkHttp
date: 2016-11-27 18:14:43
tags: [Java, Util]
---

> 在 Java 平台上，我们一般使用 Apache HttpClient 作为通常的 HTTP 客户端。Square 公司开源的 OkHttp 是一个更先进的专注于连接效率的 HTTP 客户端。OkHttp 提供了对 HTTP/2 和 SPDY 的支持，并提供了连接池，GZIP 压缩和 HTTP 响应缓存功能。OkHttp 的 API 接口也更加的简单实用。可以将 OkHttp 作为 Apache HttpClient 的升级与替换，本文将对其进行详细的介绍，更详细内容请参考 <https://www.ibm.com/developerworks/cn/java/j-lo-okhttp>

<!--more-->

## OkHttp

### Gradle 依赖

```groovy
compile 'com.squareup.okhttp3:okhttp:3.4.2'
```

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

但是如果用上面的代码访问 https 的网站如 https://www.baidu.com 则会报错，因为 https 需要加入证书信息，可以使用 OkHttp 的封装 `EasyOkHttp` 可以简化很多

## EasyOkHttp

### Gradle 依赖

```groovy
compile 'com.mzlion:easy-okhttp:1.0.7-beta'
```

```java
import com.mzlion.easyokhttp.HttpClient;

public class OkHttpEasyTest {
    public static void main(String[] args) {
        String responseData = HttpClient.get("https://www.baidu.com").execute().asString(); // 自动处理 CA 签发的证书
        System.out.println(responseData);
    }
}
```

> HttpClient 请求 https 默认需要 CA 颁发的证书，如果是自己签发的证书则请求会失败，可以忽略证书签发，也可以把在请求时传入证书:
>
> * 忽略证书:
>
>   ```java
>   String responseData = HttpClient.get("https://www.xtuer.com/").https().execute().asString(); // https() 没有参数
>   ```
>
> * 指定证书:
>
>   ```java
>   InputStream sslInput = new FileInputStream("/Users/Biao/Documents/workspace/Java/https-cert/server.crt");
>   String responseData = HttpClient.get("https://www.xtuer.com/").https(sslInput).execute().asString(); // https() 的参数为证书的 input stream
>   ```

## 参考资料

* [EasyOkHttp 帮助文档](https://www.w3cschool.cn/easyokhttp)
* [对 OkHttp 网络框架的封装 EasyOkHttp](http://www.oschina.net/p/easy-okhttp?fromerr=vg7BzejC)

