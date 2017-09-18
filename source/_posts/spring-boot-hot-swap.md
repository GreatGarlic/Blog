---
title: Spring Boot 热更新
date: 2017-09-12 10:46:34
tags: SpringBoot
---

热更新在开发中对于提高效率是非常重要的，SpringBoot 带了一个 `org.springframework.boot:spring-boot-devtools`，但是在 SpringBoot + Gradle + IDEA 的配合中不怎么好用，下面介绍另一种热更新的方式:

* 引入 springloaded(不需要配置其他的):

  ```groovy
  buildscript {
      repositories {
          jcenter()
      }
      dependencies {
          classpath 'org.springframework:springloaded:1.2.8.RELEASE'
      }
  }
  ```

* 终端进入项目目录，执行 `gradle -t classes` 启动一个监听任务，当发现项目中的 Java 类发生变化时进行自动编译，模版文件变化时自动复制到 build 对应的目录中

* 终端进入项目目录，执行 `gradle bootRun` 启动项目

  > 修改 Java 文件和模版文件等看看效果

