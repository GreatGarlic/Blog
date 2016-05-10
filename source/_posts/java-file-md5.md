---
title: 计算文件的 MD5
date: 2016-05-08 22:06:05
tags: [Java, Spring]
---

使用 Spring core 的 `DigestUtils` 计算文件的 MD5。

<!--more-->

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

import org.springframework.util.DigestUtils;

public class CommonUtils {
    /**
     * 计算文件的 MD5.
     *
     * @param filePath 文件的路径
     * @return 文件的 MD5
     */
    public static String fileMd5(String filePath) {
        InputStream in = null;

        try {
            in = new FileInputStream(filePath);
            return DigestUtils.md5DigestAsHex(in); // Spring 自带的 MD5 工具
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        return "";
    }
    
    public static void main(String[] args) {
        System.out.println(fileMd5("/Users/Biao/Desktop/x.rar"));
    }
}
```
