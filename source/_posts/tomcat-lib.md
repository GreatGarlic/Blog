---
title: Tomcat 自定义 lib 目录
date: 2017-03-30 13:51:38
tags: Java
---

现在微服务挺流行的，以前的一个项目使用微服务后可能被拆开成了 5 个小项目，但是有的公司服务器不够，很可能把其中几个项目都安装在同一台机器上，甚至这些项目都使用同一个 Tomcat 来运行。

现在假设这些项目都使用 SpringMvc 来开发，以前一个项目的时候，项目的 lib 里放一份 SpringMvc 的 jar 包就可以了，现在因为分成了 5 个项目，则每个项目都在自己的 lib 里带上 SpringMvc 的 jar 包，结果就是比以前多了 4 分 SpringMvc 的 jar 包。

**能不能把一份 SpringMvc 的 jar 包放在一个共同的目录下，然后这些项目都共同使用呢？**<!--more-->

可以的，例如把这些 jar 包放在目录 **/Users/Biao/Tomcat-Shared-Lib** 下，只需要把这个路径加入到 Tomcat 的 **conf/catalina.properties** 中的 **shared.loader** 或则 **common.loader** 里即可:

```
shared.loader="/Users/Biao/Tomcat-Shared-Lib","/Users/Biao/Tomcat-Shared-Lib/*.jar"
```

> 需要注意一点的是，项目中例如使用 logback, log4j 等的日志配置对共享目录下的 jar 无效，Tomcat 启动后会看到很多 SpringMvc 输出的 Debug 日志。为了对 Tomcat-Shared-Lib 中 SpringMvc 的日志进行控制，只需要把日志的配置文件也放到这个目录里就可以了。

Tomcat 定义了几个类加载器: 

```
      Bootstrap
          |
       System
          |
       Common
       /     \
  Webapp1   Webapp2 ...
```

详细的信息请参考 [Class Loader HOW-TO](http://tomcat.apache.org/tomcat-7.0-doc/class-loader-howto.html):

> In a Java environment, class loaders are arranged in a parent-child tree. Normally, **when a class loader is asked to load a particular class or resource, it delegates the request to a parent class loader first**, and then looks in its own repositories only if the parent class loader(s) cannot find the requested class or resource. Note, that the model for web application class loaders *differs* slightly from this, as discussed below, but the main principles are the same.
>
> When Tomcat is started, it creates a set of class loaders that are organized into the following parent-child relationships, where the parent class loader is above the child class loader:


> **WebappX** — A class loader is created for each web application that is deployed in a single Tomcat instance. All unpacked classes and resources in the `/WEB-INF/classes` directory of your web application, plus classes and resources in JAR files under the `/WEB-INF/lib` directory of your web application, are made visible to this web application, but not to other ones.



