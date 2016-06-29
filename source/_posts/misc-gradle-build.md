---
title: Gradle build 脚本
date: 2016-06-29 10:20:00
tags: [Misc, Gradle]
---

单模块和多模块 Gradle 的构建脚本样例。

<!--more-->

## 单模块 `build.gradle`

```java
group 'com.xtuer'
version '1.0-SNAPSHOT'

apply plugin: 'java'
apply plugin: 'maven'

/* 解决设置版本不起作用问题 */
tasks.withType(JavaCompile) {
    sourceCompatibility = '1.8'
    targetCompatibility = '1.8'
}

[compileJava, compileTestJava, javadoc]*.options*.encoding = 'UTF-8'

////////////////////////////////////////////////////////////////////////////////
//                                   Maven 依赖                               //
////////////////////////////////////////////////////////////////////////////////
repositories {
    mavenLocal()
    mavenCentral()
}

ext {
    springVersion = '4.1.1.RELEASE'
    springIntegrationVersion = '4.2.4.RELEASE'
}

dependencies {
    compile(
            "org.springframework:spring-webmvc:$springVersion",
            "org.springframework.integration:spring-integration-core:$springIntegrationVersion"
    )

    testCompile "junit:junit:4.12"
}
```

## 多模块 `build.gradle`

```java
allprojects {
    group 'com.xtuer'
    version '1.0-SNAPSHOT'

    apply plugin: 'java'
    apply plugin: 'maven'

    /* 解决设置版本不起作用问题 */
    tasks.withType(JavaCompile) {
        sourceCompatibility = '1.8'
        targetCompatibility = '1.8'
    }
    
    [compileJava, compileTestJava, javadoc]*.options*.encoding = 'UTF-8'
}

subprojects {
    repositories {
        mavenLocal()
        mavenCentral()
    }

    ext {
        springVersion = '4.1.1.RELEASE'
        springIntegrationVersion = '4.2.4.RELEASE'
    }

    dependencies {
        compile(
                "org.springframework:spring-webmvc:$springVersion",
                "org.springframework.integration:spring-integration-core:$springIntegrationVersion"
        )

        testCompile "junit:junit:4.12"
    }
}
```

> `settings.gradle`: include 包含子模块的名字

```
rootProject.name = 'SpringIntegration'
include 'Tutorial'
```
