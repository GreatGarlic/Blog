---
title: Gradle 编码
date: 2016-04-15 13:12:58
tags: Gradle
---

Gradle 默认使用系统字符编码(Windows 为 GBK，Linux, Mac 为 UTF-8)，很多程序员都是使用 Windows，但是 Java 文件以及其他资源文件大多数都会使用 UTF-8(因为要跨平台使用)，在 Windows 开发时编译运行容易出现乱码，报错等。

<!--more-->

build.gradle 中一般会使用下面的设置来设定编码

```
[compileJava, compileTestJava, javadoc]*.options*.encoding = 'UTF-8'
```

但是这个设置只是解决了 Java 源码和文档的编码，如果资源文件中有中文，里面有占位符，需要 Gradle 动态替换生成，新生成的资源文件中就会出现乱码。

为了彻底的解决乱码问题，可以设置 Gradle 运行环境的编码为 UTF-8，有 2 种方法

* 项目的 `gradle.properties` 里添加下面的代码，只会影响项目自己

    ```
    systemProp.file.encoding=UTF-8
    ```

* `<gradle>/bin/grable.bat` 里第 12 行左右设置 JVM 编码为 UTF-8，会影响所有项目

    ```
    set DEFAULT_JVM_OPTS="-Dfile.encoding=UTF-8"
    ```

哪一种方式更好？如果可能，推荐所有编码都使用 UTF-8 的方式。

