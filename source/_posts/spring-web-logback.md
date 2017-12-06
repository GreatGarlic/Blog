---
title: 使用 Logback 记录日志
date: 2016-10-15 15:48:45
tags: SpringWeb
---
Logback 和 Log4j 是同一个人开发的，Logback 比 Log4j 的功能更强大，效率更高，但配置几乎一样。

<!--more-->

## Gradle 依赖
```groovy
compile 'ch.qos.logback:logback-classic:1.1.3'
```

## 把 logback.xml 放到 resources 目录里
```xml
<?xml version="1.0"?>
<configuration scan="true" scanPeriod="30 seconds">
    <property name="logDir" value="/temp/logs"/>

    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss}] [%-5level] [%F-%M:%L] - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss}] [%-5level] [%F-%M:%L] - %msg%n</pattern>
        </encoder>

        <file>${logDir}/log.txt</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logDir}/log_%d{yyyyMMdd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>

    <root level="info">
        <appender-ref ref="stdout"/>
        <appender-ref ref="file"/>
    </root>

    <logger name="org.apache.ibatis.io" level="off"/>
    <logger name="org.springframework"  level="info"/>
    <logger name="org.mybatis"          level="off"/>
    <logger name="com.xtuer.mapper"     level="off"/>
</configuration>
```

## 按小时生成日志
上面的配置 logback 是按天生成日志文件的，如果在访问量大的系统，需要按小时生成日志文件，则只需要修改 fileNamePattern 即可，例如 

```
<fileNamePattern>${log.base}/log_%d{yyyy-MM-dd_HH}.log</fileNamePattern>
```
`HH` 表示按小时生成

## 按大小生成日志

把 appender 换为下面的配置即可

```xml
<appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${log.base}/log.txt</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <!-- rollover daily -->
        <fileNamePattern>${log.base}/log_%d{yyyyMMdd}.%i.log</fileNamePattern>
        <!-- each file should be at most 500MB, keep 30 days worth of history, but at most 30GB -->
        <maxFileSize>500MB</maxFileSize>
        <maxHistory>30</maxHistory>
        <totalSizeCap>30GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>[%d{yyyy-MM-dd HH:mm:ss}] [%-5level] [%F-%M:%L] - %msg%n</pattern>
    </encoder>
</appender>
```

## 使用 logback 输出日志

通过前面 2 步，logback 就配置好了，接下来就可以在代码里像下面这么使用 logback。

```java
package com.xtuer.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class DemoController {
    // 1. 创建 logger 对象
    private static Logger logger = LoggerFactory.getLogger(DemoController.class);

    @GetMapping("/logback")
    @ResponseBody
    public String logback() {
        // 2. 和 log4j 一样使用
        logger.debug("No params");

        // 3. 可以使用 {} 的方式传入参数
        logger.debug("With params: time: {}, name: {}", System.nanoTime(), "Bingo");

        return "Test logback";
    }
}
```

终端输出:

```
[2016-10-15 16:03:55] [DEBUG] [DemoController.java-logback:20] - No params
[2016-10-15 16:03:55] [DEBUG] [DemoController.java-logback:23] - With params: time: 460693246578509, name: Bingo
```

同时在 `/temp/logs` 生成了日志文件 `log.txt`

## 使用 Logback 输出 Spring 的日志

由于历史原因，Spring 的日志使用的是 JCL，而现在 SLF4J 更流行，Logback 也是在 SLF4J 的基础上输出的，为了把 JCL 的日志桥接到 SLF4J，即使用 Logback 输出，只需要把下面的依赖添加到项目即可，不需要修改其他地方，这样 Spring 的日志会自动的使用 Logback 输出了

```xml
compile 'org.slf4j:jcl-over-slf4j:1.7.25'
```

`参考文档`：<http://unmi.cc/jcl-over-slf4j-slf4j/>

## Logback 的日志级别：

1. `ALL`: 是最低等级的，用于打开所有日志记录。 
2. `DEBUG`: 指出细粒度信息事件对调试应用程序是非常有帮助的。
3. `INFO`: 表明消息在粗粒度级别上突出强调应用程序的运行过程。 
4. `WARN`: 表明会出现潜在错误的情形。
5. `ERROR`: 指出虽然发生错误事件，但仍然不影响系统的继续运行。
6. `FATAL`: 指出每个严重的错误事件将会导致应用程序的退出。
7. `OFF`: 是最高等级的，用于关闭所有日志记录。

## 优先级
`优先级`从低到高分别是 ALL、DEBUG、INFO、WARN、ERROR、FATAL、OFF

通过定义级别，您可以控制到应用程序中相应级别的日志信息的开关。比如在这里定义了 INFO 级别，则应用程序中所有 DEBUG 级别的日志信息将不被打印出来。`程序会打印高于或等于所设置级别的日志`，设置的日志等级越高，打印出来的日志就越少。如果设置级别为 INFO，则优先级高于等于 INFO 级别（如：INFO、 WARN、ERROR, FATAL）的日志信息将可以被输出，小于该级别的如 DEBUG 将不会被输出。

## 过滤器
Logback 还支持`过滤器`，例如将过滤器的日志级别配置为 ERROR，所有 ERROR 级别的日志交给 appender 处理，非 ERROR 级别的日志，被过滤掉。过滤器被添加到 appender 中，为 appender 添加一个或多个过滤器后，可以用任意条件对日志进行过滤。appender 有多个过滤器时，按照配置顺序执行。通过 appender 中的 filter 来严格限制日志的输出级别：

```xml
<filter class="ch.qos.logback.classic.filter.LevelFilter">
	<level>ERROR</level>
	<onMatch>ACCEPT</onMatch>
	<onMismatch>DENY</onMismatch>
</filter>
```

## 精确控制日志的应用范围
在程序调试中，经常出现的情况是：错误只在某一个或者几个类或者包里，所以只需要打开这几个类或者包里的 DEBUG 级别的 log。在以前的项目，使用 Spring 和 Hibernate 时，一旦打开 DEBUG 级别的 log，程序本身的 debug 信息就会被 Spring 和 Hibernate 的大量日志淹没，大大降低了调试的效率。而 logback 让这一切变的简单起来了：

```xml
<logger name="org" level="ERROR"/>
```

这一行就将org包下面的所有日志级别设为了ERROR，不会再打扰我们的 DEBUG。

## 参考文档：

<http://logback.qos.ch/manual/appenders.html#SizeAndTimeBasedFNATP>
<http://blog.csdn.net/mydeman/article/details/6716925>
<http://www.360doc.com/content/12/0321/13/203871_196275021.shtml>

