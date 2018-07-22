---
title: Office 文档转为 PDF 和 HTML
date: 2017-05-25 10:29:58
tags: [Java, Util]
---

下面介绍使用 JodConverter + LibreOffice 把 Windows Office 的 doc，docx，xls 等文档转换为 PDF 和 HTML:

* HTML:
  * 优点:  用浏览器打开方便，便于实现 doc 等在线预览
  * 缺点: 相对于 PDF 大不少，图片是独立文件，格式也没有 PDF 的漂亮
* PDF:
  * 优点: 比 HTML 格式小，格式比较接近于原文档
  * 缺点: 相对于 HTML 在线预览不够方便，也可以借助 pdf.js + HTML5 实现[在线预览](/pdf-online/)

<!--more-->

## 安装 LibreOffice

下载 LibreOffice <http://www.libreoffice.org/download/download/> 然后安装到默认目录

## Gradle 依赖

```groovy
compile 'org.jodconverter:jodconverter-spring:4.2.0'
```

## Spring 配置文件

```xml
<!--文件名: beans.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="converterBean" class="org.jodconverter.spring.JodConverterBean">
        <!--<property name="officeHome" value=""/>-->
    </bean>
</beans>
```

> Linux 和 Windows 需要设置 officeHome，即 LibreOffice 的安装目录。

## 转换代码

```java
import org.jodconverter.DocumentConverter;
import org.jodconverter.office.OfficeException;
import org.jodconverter.spring.JodConverterBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import java.io.File;

public class OfficeConverter {
    public static void main(String[] args) throws OfficeException {
        // 程序启动的时候创建 converterBean 对象，然后程序中使用它的 converter 进行转换
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        JodConverterBean converterBean = context.getBean("converterBean", JodConverterBean.class);

        // DocumentConverter 支持把 doc, docx, xls 等 office 文档转换为 pdf, html
        // 把 doc 转换为 pdf，目标格式根据输出文件名的后缀自动决定
        DocumentConverter converter = converterBean.getConverter();
        converter.convert(new File("/Users/Biao/Desktop/使用说明.doc")).to(new File("/Users/Biao/Desktop/out/使用说明.pdf")).execute();
        converter.convert(new File("/Users/Biao/Desktop/使用说明.doc")).to(new File("/Users/Biao/Desktop/out/使用说明.html")).execute();
    }
}
```

## 运行

运行程序会自动的打开一个 LibreOffice 后台服务，默认端口是 2002，然后进行转换。

> 如果输出文件的目录不存在，会自动创建的。
>
> 启动 LibreOffice，连接到 LibreOffice 的时间会比较长，差不多需要 30s，第一次转换执行后的转换都非常快，普通文档转换 1s-2s 就够了。
>
> 如果想看到日志，需要配置 logback 或者 log4j。

## 思考

上面的代码依赖了 Spring 4，是为了能够方便的使用配置的方式让 Spring IoC 容器管理对象，如果想要脱离 Spring，或则用的 Spring 3，那么只需阅读一下 **org.jodconverter.spring.JodConverterBean** 的代码，然后相应的修改一下。

JodConverterBean 可以配置好多参数，例如 LibreOffice 不是使用默认目录安装的，则需要配置它的 officeHome 属性，否则服务启动不成功，具体查看其属性就知道能配置什么了。

Spring 下的代码更多是为了和 Web 一起使用，如果只是本地进行转换，例如有一批 word 要转 pdf，可以像下面这样简单使用

```java
import org.jodconverter.JodConverter;
import org.jodconverter.office.LocalOfficeManager;
import org.jodconverter.office.LocalOfficeUtils;
import org.jodconverter.office.OfficeManager;

import java.io.File;

public class ConverterInLocal {
    public static void main(String[] args) throws Exception {
        OfficeManager officeManager = LocalOfficeManager.install();

        try {
            officeManager.start();

            File src = new File("/Users/Biao/Desktop/使用说明.doc");
            File dst = new File("/Users/Biao/Desktop/out/使用说明.html");
            JodConverter.convert(src).to(dst).execute();
        } finally {
            LocalOfficeUtils.stopQuietly(officeManager);
        }
    }
}
```

## 参考资料

* [JodConverter Github](https://github.com/sbraconnier/jodconverter)
* [JodConverter Wiki](https://github.com/sbraconnier/jodconverter/wiki)
* [JodConverter 4.1.0 版本改进解析](https://segmentfault.com/a/1190000011657122)