---
title: Gradle 创建 App Module
date: 2016-06-29 10:38:57
tags: [Misc, Gradle]
---

> 打包需要执行 `gradle shadowJar`，参考 <https://github.com/johnrengelman/shadow>  
> 如果不使用 `shadowJar` 打包的话，由于 Spring 的 schema 都分散到不同文件里了，META-INF 中的 schema 文件后面的会覆盖前面的导致运行出错，例如使用了 `context:component-scan` 就会出错。
> 
> 下面的配置支持 Spring + MyBatis

<!--more-->

```java
group 'com.xtuer'
version '1.0'

apply plugin: 'java'
apply plugin: 'maven'
apply plugin: 'application'
apply plugin: 'com.github.johnrengelman.shadow'

/* 打 Jar 包 */
buildscript {
    repositories { jcenter() }
    dependencies { classpath 'com.github.jengelman.gradle.plugins:shadow:1.2.2' }
}

mainClassName = 'Main'

jar {
    manifest { attributes 'Main-Class': mainClassName }
}

shadowJar {
    mergeServiceFiles('META-INF/spring.*')
}

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
    mysqlVersion = '5.1.21'
    springVersion = '4.1.1.RELEASE'
    mybatisVersion = '3.2.1'
    mybatisSpringVersion = '1.2.2'
    dbcpVersion = '1.4'
}

dependencies {
    compile(
            "org.springframework:spring-webmvc:$springVersion",
            "org.springframework:spring-jdbc:$springVersion",
            "mysql:mysql-connector-java:$mysqlVersion",
            "org.mybatis:mybatis-spring:$mybatisSpringVersion",
            "org.mybatis:mybatis:$mybatisVersion",
            "commons-dbcp:commons-dbcp:$dbcpVersion"
    )

    testCompile "junit:junit:4.12"
}
```
