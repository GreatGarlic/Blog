---
title: 使用百度 OCR 服务识别图片中的文本
date: 2018-02-07 11:19:43
tags: Java
---

访问 <https://cloud.baidu.com/product/ocr/general> 可以先体验一下百度的 OCR 文字识别，在`功能演示`处上传一个含有文字的图片就可以看到识别效果，还是挺不错的，接下来就介绍使用 OCR 服务的编程实现:

1. 点击`立即使用`

2. 点击`创建应用` (需要登陆)

3. 得到应用 `API Key` 和 `Secret Key` (在程序中需要使用，对应程序中的 `APP_ID` 和 `APP_KEY`)

4. 使用 `API Key` 和 `Secret Key` 换取 `access_token`，请参[考鉴权认证机制](http://ai.baidu.com/docs#/Auth/top)

5. 使用 OCR 服务识别图片中的文字，请参考[通用文字识别](https://cloud.baidu.com/doc/OCR/OCR-API.html#.E8.AF.B7.E6.B1.82.E8.AF.B4.E6.98.8E)

   * 把图片进行 Base64 编码成为字符串

     > 文档中说**所有图片均需要 Base64 编码后再进行 urlencode**，这里容易造成困扰，`其实 Base64 后就够了`，因为 Base64 包含的 64 个字符为 `a-z`, `A-Z`, `0-9`, `/`, `+` 以及填充字符 `=` 都包含在了 urlencode 不需要进行编码的字符内。

   * 去掉图片头，如（data:image/jpg;base64,）

   * 传给百度，然后就能得到 JSON 结果<!--more-->

下面给出 Java 的实现以供参考:

```java
import com.alibaba.fastjson.JSONObject;
import com.mzlion.easyokhttp.HttpClient;

import java.io.IOException;

/**
 * 使用百度的 OCR 服务识别图片中的文字
 */
public class BaiduOcr {
    private static final String APP_ID  = "k364IHWiCdW1gZtL6eKfNRqM";
    private static final String APP_KEY = "2sGL1HlcoYDaStLiCrsEiNRqHbDQEWax";
    private static final String TOKEN   = "24.2f86da893aacea8f1af0063ccdf02858.2592000.1520561223.282335-10804628"; // 30 天有效期

    public static void main(String[] args) throws IOException {
        String json = detectText("/Users/Biao/Desktop/y.png");
        System.out.println(json);
    }

    // 识别图中的文字，返回 JSON 格式字符串, path 为图片的路径
    public static String detectText(String path) throws IOException {
        String image = ImageBase64Utils.imageToBase64String(path);
        int startIndex = image.indexOf(",") + 1;
        image = image.substring(startIndex); // 图片的 Base64 编码是不包含图片头的，如（data:image/jpg;base64,）

        String response = HttpClient.post("https://aip.baidubce.com/rest/2.0/ocr/v1/general_basic")
                .param("access_token", TOKEN)
                .param("image", image)
                .execute().asString();
        return response;
    }

    // 使用 APP_ID + APP_KEY 换取 token
    public static String requestToken() {
        String response = HttpClient.post("https://aip.baidubce.com/oauth/2.0/token")
                .param("grant_type", "client_credentials")
                .param("client_id", APP_ID)
                .param("client_secret", APP_KEY)
                .execute().asString();
        JSONObject tokenObject = JSONObject.parseObject(response);
        return tokenObject.get("access_token").toString();
    }
}
```

> 依赖了下面的库:
>
> * fastjson: `compile 'com.alibaba:fastjson:1.2.21'`
> * easy-okhttp: `compile 'com.mzlion:easy-okhttp:1.1.2'`
> * ImageBase64Utils: <http://qtdebug.com/java-image-base64>