---
title: 签名验证
date: 2017-07-13 11:22:33
tags: [Java, Util]
---

## 签名验证

签名验证涉及到客户端（比如一个 Web 应用）和服务器端，每个客户端在服务器上有一个对应的 **app_id** 和 **app_key**，大致步骤如下：

1. 客户端使用 app_id + app_key + `其他参数`生成签名字符串 sign

2. 把 app_id、`其他参数` 和 sign 一起发送给服务器（app_key 不发送）

3. 服务器接收到请求后，根据参数中的 app_id 查找到对应的 app_key，然后根据签名算法生成签名字符串 sign2

   > 客户端和服务器端使用同样的签名算法生成签名字符串。

4. 字符串比较参数中的 sign 和服务器生成的 sign2，如果相等则签名没问题，放行访问，否则签名无效，拒绝访问<!--more-->

## 签名算法

1. 设要参与计算签名的数据为集合 M，将集合 M 内非空参数值的参数按照参数名 ASCII 码从小到大排序（字典序），使用 URL 键值对的格式（即key1=value1&key2=value2…）拼接成字符串 signTemp。

   > 特别注意以下重要规则：
   >
   > * 参数名 ASCII 码从小到大排序（字典序）
   > * 如果参数的值为空不参与签名
   > * 参数名区分大小写
   > * 验证调用返回或主动通知签名时，传送的 sign 参数不参与签名，将生成的签名与该 sign 值作校验
   > * 接口可能增加字段，验证签名时必须支持增加的扩展字段

2. 对 signTemp 进行 MD5 运算，再将得到的字符串所有字符转换为大写，得到 sign 值 signValue。

## 例子

假设传送的参数如下：

```js
app_id: 015B512C873648578FB2C32BD5677BD4
username: alice
productId: 1001
signedTime: 1499914521231

并且 app_key 为 927170905ECA42FC9813DD7EED21A5AF
```

第一步：对参数按照 key=value 的格式，并按照参数名 ASCII 字典序排序：

```js
signTemp = "app_id=015B512C873648578FB2C32BD5677BD4&app_key=927170905ECA42FC9813DD7EED21A5AF&productId=1001&signedTime=1499914521231&username=alice";
```

第二步：计算 signTemp 的签名字符串：

```js
sign = md5(signTemp).toUpperCase(); // 281879C9007C3698D1106F9CF6A097A3
```

第三步：发送参数（没有 app_key）

```
app_id=015B512C873648578FB2C32BD5677BD4&productId=1001&signedTime=1499914521231&username=alice&sign=33A62BBCEF9D4AF675ADC6BAEA468B99
```

## Java 实现

```java
import org.springframework.util.DigestUtils;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.TreeMap;

public class Sign {
    public static void main(String[] args) throws IOException {
        String appId = "015B512C873648578FB2C32BD5677BD4";
        String appKey = "927170905ECA42FC9813DD7EED21A5AF";

        // 参与签名的参数
        Map<String, String> params = new HashMap<>();
        params.put("app_id", appId);
        params.put("app_key", appKey);
        params.put("username", "alice");
        params.put("productId", "1001");
        params.put("signedTime", "1499914521231");

        String signValue = sign(params);
        System.out.println(signValue); // 281879C9007C3698D1106F9CF6A097A3
    }

    public static String sign(Map<String, String> params) {
        Map<String, String> temp = new TreeMap<>(params); // 对参数进行排序

        // 拼接参数
        StringBuilder sb = new StringBuilder();
        for (String key : temp.keySet()) {
            sb.append(key).append("=").append(temp.get(key)).append("&");
        }

        String signTemp = sb.deleteCharAt(sb.length() - 1).toString(); // 去掉最后一个 &
        String signValue = DigestUtils.md5DigestAsHex(signTemp.getBytes()).toUpperCase(); // 使用 MD5 计算签名字符串

        return signValue;
    }
}
```

## 签名有效期

如果不限制签名的使用时间，则生成的签名永远有效，如果别人拿到生成的链接后就能一直使用了，这是比较危险的，所以限制签名时长很有必要，有 2 种方式可以限制签名的有效时间：

1. 客户端加签名生成时间：

   1. 生成签名时的时间加入计算签名

   2. 服务器收到请求后：`签名有效` 并且 `服务器当前时间 <= 参数中的时间 + 过期时长` 则放行访问，否则拒绝访问

      > * 如果客户端的时间或则服务器端的时间不准，就有可能签名会无效，例如客户端的时间是 2016 年的，而服务器端的是 2017 年的，签名的过期时长为 5 分钟，则计算的签名就会是无效的。反过来签名的有效时间就变长了。
      >
      >
      > * 还有个办法，使用服务器时间进行签名：在计算签名前先向服务器请求一下服务器的时间，然后用此时间进行签名即可。服务器接收到请求时判断一下这个时间一定要比服务器当前时间早才行。

2. 服务器记录签名时间：

   1. 客户端计算签名（不使用时间）

   2. 服务器收到请求后：

      * 如果签名有效且是第一次使用，则保存签名和时间 **T1** 到数据库或则 Redis 等，放行访问
      * 如果签名不是第一次访问（能从数据库或则 Redis 中查找到），并且 `服务器当前时间 <= T1 + 过期时长` 则放行访问，否则拒绝访问

      > 优点：此种方式是最安全的。
      >
      > 缺点：服务器端需要一直保存签名，否则删除后再次使用时就变成第一次使用了，不过 Redis 之类的存储几亿条记录访问还是很快的。