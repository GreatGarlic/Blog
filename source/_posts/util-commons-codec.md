---
title: Commons-Codec 例子
date: 2016-05-22 10:11:29
tags: [Java, Util]
---

可以使用 Apache Commons-Codec 计算:

* 字符串的 MD5 (SHA)
* 文件的 MD5 (SHA)
* Base64 编码
* Base64 解码

<!--more-->

## Gradle 依赖
```groovy
compile 'commons-codec:commons-codec:1.10'
```

## 计算字符串的 MD5
```java
import org.apache.commons.codec.digest.DigestUtils;

public class Test {
    public static void main(String[] args) {
        System.out.println(DigestUtils.md5Hex("Hello"));
    }
}
```

## 计算文件的 MD5
```java
import org.apache.commons.codec.digest.DigestUtils;

import java.io.FileInputStream;
import java.io.IOException;

public class Test {
    public static String fileMd5(String path) {
        try {
            return DigestUtils.md5Hex(new FileInputStream(path));
        } catch (IOException e) {
            e.printStackTrace();
        }

        return "";
    }

    public static void main(String[] args) {
        System.out.println(fileMd5("/Users/Biao/Desktop/biao.png")); // 输出: 7d3caca345a66833248c627d1eb7fb54
    }
}
```

## Base64 编码解码
```java
import org.apache.commons.codec.binary.Base64;

public class Test {
    public static void main(String[] args) throws Exception {
        Base64 base64 = new Base64(); // 构造函数里可以定义输出时每一行的最大长度, 默认没有限制

        // Base64 encode
        String result = base64.encodeToString("Hello".getBytes("UTF-8"));
        System.out.println(result); // 输出: SGVsbG8=

        // Base64 decode
        result = new String(base64.decode(result));
        System.out.println(result); // 输出: Hello
    }
}
```
