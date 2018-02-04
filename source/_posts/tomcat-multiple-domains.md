---
title: Tomcat 多域名
date: 2017-03-29 22:50:51
tags: Java
---

一个 Tomcat 可以支持多个域名，需要在 server.xml 中为每一个域名配置一个 **Host**，参考默认的 localhost 的配置。

安装 Tomcat 后，在 **conf/server.xml** 中默认有一个 localhost 的 Host:

```xml
<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="localhost_access_log" suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b"/>
</Host>
```

**术语:**

* 应用：一个域名对应一个 Web 应用
* 项目：一个域名下可以有多个项目，参考 Tomcat webapps 下的多个项目，例如 ROOT，manager，examples

<!--more-->

## 配置新域名

一、有一个新的域名 www.foo.com，则在 server.xml 中添加 Host 如下:

```xml
<Host name="www.foo.com" appBase="/Users/Biao/Desktop/www.foo.com" unpackWARs="true" autoDeploy="true">
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="/Users/Biao/Desktop/www.foo.com/logs" prefix="foo_access_log" suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b"/>
</Host>
```

* name 即新的域名

* appBase 为新域名对应的 web 应用的根目录，如果不是绝对路径，则是相对于 Tomcat 的 Home 目录

  > 目录名使用 www.foo.com，是为了表示对应此域名，目的明确，当然也可以用任意的名字，例如 fooox

* Valve 是配置日志，directory 是相对路径时相对于 Tomcat 的 Home 目录，而不是对应的应用目录

二、在 /Users/Biao/Desktop/www.foo.com 目录下创建一个 ROOT 目录，把项目的文件放在 ROOT 目录下:

```
├── www.foo.com
│   ├── ROOT
│   │   ├── style.css
│   │   └── test.html
│   └── logs
│       └── foo_access_log.2017-03-29.txt
```

三、访问 <http://www.foo.com:8080/test.html> 则看到了新域名生效了。

> **提示:** 测试阶段可以把 www.foo.com 添加到 host 中，例如 127.0.0.1 www.foo.com
>

**为啥项目的文件要放在 ROOT 目录下？**

> www.foo.com 目录就相当于 webapps 目录，这么理解是不是就明白了？

**怎么在域名 www.foo.com 下增加一个新的项目?**

> 例如项目 pandora，放在 www.foo.com 目录下还是其他地方，访问它的 URL 是不是 <http://www.foo.com/pandora>？

尝试理解下下面的 Host 的意思:

```xml
<Host name="www.bar.com" appBase="bar" unpackWARs="true" autoDeploy="true">
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="bar_access_log" suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b"/>
</Host>
```

## 默认 Host

```xml
<Engine name="Catalina" defaultHost="localhost">
```

**defaultHost** 是指使用 IP 访问时 Tomcat 默认用来处理请求的 Host，也就是访问此域名对应的应用。如果有多个域名时，最好是配置一下。

## Host 别名

可以为同一个应用配置多个域名:

```xml
<Host name="www.foo.com" appBase="/Users/Biao/Desktop/www.foo.com" unpackWARs="true" autoDeploy="true">
    <Alias>foo.com</Alias>
    <Alias>sub.foo.com</Alias>
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="/Users/Biao/Desktop/www.foo.com/logs" prefix="foo_access_log" suffix=".txt" pattern="%h %l %u %t &quot;%r&quot; %s %b"/>
</Host>
```

www.foo.com，sub.foo.com 和 foo.com 都指向同一个应用。