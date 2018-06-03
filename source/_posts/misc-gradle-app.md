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
plugins {
    id 'java'
    id 'application'
    id 'com.github.johnrengelman.shadow' version '2.0.2'
}

////////////////////////////////////////////////////////////////////////////////
//                                [1] [2] 运行、打包                           //
////////////////////////////////////////////////////////////////////////////////
// [1.1] 从命令行运行默认类: gradle run
// [1.2] 从命令行运行某个类: gradle run -DmainClass=Foo
ext {
    project.mainClassName = System.getProperty("mainClass", "DefaultMainClass")
}

// [2] 打包: gradle clean shadowJar [-DmainClass=Foo]
shadowJar {
    mergeServiceFiles('META-INF/spring.*')
}

////////////////////////////////////////////////////////////////////////////////
//                                 [3] Maven 依赖                             //
////////////////////////////////////////////////////////////////////////////////
repositories {
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

    compileOnly 'org.projectlombok:lombok:1.16.18'
    testCompile 'junit:junit:4.12'
}

////////////////////////////////////////////////////////////////////////////////
//                                    JVM                                     //
////////////////////////////////////////////////////////////////////////////////
sourceCompatibility = JavaVersion.VERSION_1_8
targetCompatibility = JavaVersion.VERSION_1_8
[compileJava, compileTestJava, javadoc]*.options*.encoding = 'UTF-8'

tasks.withType(JavaCompile) {
    options.compilerArgs << '-Xlint:unchecked' << '-Xlint:deprecation'
}
```
