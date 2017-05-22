---
title: Java Tips
date: 2016-04-02 11:13:52
tags:  [Java, Util]
---

一些 Java 里常用的代码片段，例如 Bean to String 等。

<!--more-->

## Bean to String
使用 Apache Commons Lang3

```java
import org.apache.commons.lang3.builder.ReflectionToStringBuilder;
import org.apache.commons.lang3.builder.ToStringStyle;
import org.apache.commons.lang3.builder.ToStringBuilder;

System.out.println(ReflectionToStringBuilder.toString(obj));
System.out.println(ReflectionToStringBuilder.toString(obj, ToStringStyle.MULTI_LINE_STYLE));
System.out.println(ToStringBuilder.reflectionToString(p));
```

使用 Fastjson 格式化为 JSON 字符串更漂亮

```java
import com.alibaba.fastjson.JSON;

System.out.println(JSON.toJSONString(box));
```

或则使用 Lombok 的 @ToString 自动生成字符串也不错

## 取得当前的方法名

```java
Thread.currentThread().getStackTrace()[1].getMethodName()
```

## Spring 获取 HttpServletRequest
```java
HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
```

## SpringSecurity 获取登陆用户名

```java
String username = SecurityContextHolder.getContext().getAuthentication().getName();
```

## Servlet 获取客户端的 IP
```java
/**
 * 获取客户端的 IP
 * @param request
 * @return 客户端的 IP
 */
public static String getClientIp(HttpServletRequest request) {
    final String UNKNOWN = "unknown";

    // 需要注意有多个 Proxy 的情况: X-Forwarded-For: client, proxy1, proxy2
    // 没有最近的 Proxy 的 IP, 只有 1 个 Proxy 时是客户端的 IP
    String ip = request.getHeader("X-Forwarded-For");

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("X-Real-IP");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("Proxy-Client-IP");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("WL-Proxy-Client-IP");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("HTTP_CLIENT_IP");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("HTTP_X_FORWARDED_FOR");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getRemoteAddr(); // 没有使用 Proxy 时是客户端的 IP, 使用 Proxy 时是最近的 Proxy 的 IP
    }

    return ip;
}
```

> Nginx 的配置参考:
> ```
>    location / {
>
>       proxy_redirect   off;
>       proxy_set_header Host $host;
>       proxy_set_header X-Real-IP $remote_addr;
>       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
>       proxy_pass http://app_server;
>       add_header Cache-Control 'no-store';
>   }
> ```

> 参考 [HTTP 请求头中的 X-Forwarded-For](https://imququ.com/post/x-forwarded-for-header-in-http.html)
>
> `X-Forwarded-For` 是用于记录代理信息的，每经过一级代理(匿名代理除外)，代理服务器都会把这次请求的来源 IP 追加在 X-Forwarded-For 中，例如来自 4.4.4.4 的一个请求，header 包含这样一行 `X-Forwarded-For: 1.1.1.1, 2.2.2.2, 3.3.3.3` 代表请求由 1.1.1.1 发出，经过三层代理，第一层是 2.2.2.2，第二层是 3.3.3.3，而本次请求的来源 IP 4.4.4.4 是第三层代理。
>
> `X-Real-IP`，一般只记录`真实发出请求的客户端 IP`，上面的例子，如果配置了 X-Read-IP，将会是 X-Real-IP: 1.1.1.1 1所以 ，如果只有一层代理，这两个头的值就是一样的

## 字符串 MD5

```java
import org.springframework.util.DigestUtils;

public class Test {
    public static void main(String[] args) {
        System.out.println(DigestUtils.md5DigestAsHex("Biao".getBytes()));
    }
}
```

## 文件的 MD5

```java
import org.springframework.util.DigestUtils;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

public class Test {
    /**
     * 计算文件的 MD5.
     *
     * @param path 文件的路径
     * @return 文件的 MD5
     */
    public static String fileMd5(String path) {
        try (InputStream in = new FileInputStream(path)) {
            return DigestUtils.md5DigestAsHex(in); // Spring 自带的 MD5 工具
        } catch (IOException e) {
            e.printStackTrace();
        }

        return "";
    }

    public static void main(String[] args) {
        System.out.println(fileMd5("/Users/Biao/Desktop/x.png"));
    }
}

```

## 生成 UUID

```java
import java.util.UUID;

public static String uuid() {
    return UUID.randomUUID().toString().replace("-", "").toUpperCase();
}
```

## 读文件自动关闭流

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class Test {
    public static void main(String[] args) {
        String path = "/Users/Biao/Documents/workspace/Java/mix/src/main/java/Test.java";
        try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
            String line = null;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 定期执行任务

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class Test {
    public static void main(String[] args) {
        ScheduledExecutorService.scheduleAtFixedRate(() -> {
            System.out.println(System.currentTimeMillis() / 1000);
        }, 0, 1, TimeUnit.SECONDS);
    }
}
```

## 过滤后映射

```java
import java.util.stream.Stream;

public class Test {
    public static void main(String[] args) {
        Integer[] ns = {1, 2, 3, 4, 5}; // 不能是 int[]
        Stream.of(ns).filter(n -> n >= 3).mapToInt(n -> n * 2).forEach(System.out::println);
    }
}
```

> * 过滤: 去掉不要的
> * 映射: 每一个元素映射为对应的值，一个输入对应一个输出

## 创建线程

```java
public class Test {
    public static void main(String[] args) {
        // [1] 方法一
        new Thread(() -> System.out.println(1)).start();
        
        // [2] 方法二
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(2);
            }
        }).start();
    }
}

```

## 内存映射

```java
import java.io.FileInputStream;
import java.io.RandomAccessFile;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;

public class Test {
    public static void main(String[] args) throws Exception {
        FileInputStream in = new FileInputStream("/Users/Biao/Desktop/1.cpp");
        long size = in.available();
        FileChannel inChannel = in.getChannel();
        MappedByteBuffer inBuffer = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, size);

        RandomAccessFile out = new RandomAccessFile("/Users/Biao/Desktop/2.cpp", "rw");
        FileChannel outChannel = out.getChannel();
        MappedByteBuffer outBuffer = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, size);
        outBuffer.put(inBuffer);

        inChannel.close();
        outChannel.close();
        in.close();
        out.close();
    }
}
```

## 大小写不敏感的正则表达式

在前面加上 **(?i)** 即可

```java
String name = "names.NSF".replaceAll("(?i)\\.nsf$", ""); // name
```

## 文件路径

```java
// 说明这个 prop.properties 和类 PropUtil.class 是在同一个目录下 
PropUtil.class.getClassLoader().getResourceAsStream("prop.properties");

// 注意有个斜杠，说明是在 classpath 根目录下，eclipse 写的话一般如果是 bin 目录，netbeans 的话可能会在 build/classes 目录下   
PropUtil .class.getClassLoader().getResourceAsStream("/prop.properties");

// 这种是从 System Property 'user.dir'下读 prop.properties, 用 IDE 编写的话默认就是你的工程目录，一般来说 user.dir 是执行 java 命令所在的当前目录。 
new FileInputStream("prop.properties"); 
```

> 不存在所谓的 JAVASE 默认根路径的说法，java（无论是 J2SE 还是，J2EE, Web）中只有 classpath，看你的 java 命令怎么配置 classpath。

## 指定编译和运行时字符集

* 编译时使用参数 encoding

  ```
  javac -encoding utf-8 Test.java
  ```

* 运行时使用参数 -Dfile.encoding

  ```
  java -jar -Dfile.encoding=utf-8 Test.jar
  ```

## MessageFormat 格式化数字和日期

```java
import java.text.MessageFormat;

public class Test {
    public static void main(String[] argv) throws Exception {
        Object[] params = { new Integer(123), new Double(1222234.567) };
        String msg = MessageFormat.format("{0,number,percent} and {1,number,,###.##}", params);
        System.out.println(msg); // 12,300% and 1,222,234.57
    }
}
```

## SpringSecurity iframe

SpringSecurity 默认不让 iframe 嵌入当前站点的网页的，连同域都不允许，可以配置 http 让其允许

```xml
<http auto-config="true">
    <headers>
        <frame-options policy="SAMEORIGIN"/>
    </headers>
</http>
```

