---
title: 使用 Velocity 生成静态页面
date: 2017-06-30 15:47:31
tags: [Java, Util]
---

Velocity 可以作为 SpringMVC 的 View 使用，也可以用来生成邮件，静态页面等。

## Gradle 依赖

```groovy
compile 'org.apache.velocity:velocity:1.7'
compile 'org.apache.velocity:velocity-tools:2.0'
```

<!--more-->

## VelocityTest

```java
import org.apache.velocity.Template;
import org.apache.velocity.VelocityContext;
import org.apache.velocity.app.Velocity;
import org.apache.velocity.app.VelocityEngine;
import org.junit.Test;

import java.io.StringWriter;
import java.io.Writer;
import java.util.Properties;

public class VelocityTest {
    @Test
    public void generate() {
        // [1] 配置 Velocity
        String templateDirectory = "/Users/Biao/view/";
        Properties properties = new Properties();
        properties.setProperty(VelocityEngine.FILE_RESOURCE_LOADER_PATH, templateDirectory);
        properties.setProperty(VelocityEngine.INPUT_ENCODING, "UTF-8");
        properties.setProperty(VelocityEngine.OUTPUT_ENCODING, "UTF-8");
        Velocity.init(properties);

        // [2] 读取模版文件
        Template template = Velocity.getTemplate("hello.vm");

        // [3] 准备模版使用的数据
        VelocityContext ctx = new VelocityContext();
        ctx.put("name", "Hello");
        ctx.put("static", "static path");

        // [4] 使用模版和数据生成静态内容，可以用来实现页面静态化，放在 Redis 或则 Nginx 下
        Writer writer = new StringWriter();
        template.merge(ctx, writer);

        System.out.println(writer);
    }
}
```

输出:

```
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>REST</title>
</head>

<body>
    Hello 你好!<br>
    不存在的属性:  $var2<br>
    全局变量: static path<br>
    Context Path: $request.contextPath --
</body>

</html>
```

## hello.vm

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>REST</title>
</head>

<body>
    $name 你好!<br>
    不存在的属性: $!var1 $var2<br>
    全局变量: $static<br>
    Context Path: $request.contextPath --
</body>

</html>
```

