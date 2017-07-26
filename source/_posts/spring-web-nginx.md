---
title: Nginx 负载均衡
date: 2017-03-31 19:18:23
tags: Spring-Web
---

通过简单的配置，Nginx 可以实现反向代理，负载均衡，动静分离，URL 重写等。

## 代理

* 正向代理：客户使用代理访问多个外部 Web 服务器，就是翻墙
* 反向代理：多个客户使用它访问内部 Web 服务器 <!--more-->

## 负载均衡

* 七层负载均衡: Nginx  
  ![](/img/spring-web/Balance-Loader-1.jpg)  
  ![](/img/spring-web/Balance-Loader-4.png)

* 四层负载均衡: F5  
  ![](/img/spring-web/Balance-Loader-3.jpg)

* 四层和七层负载均衡的区别

  ![](/img/spring-web/Balance-Loader-5.png)

  可参考 <http://kb.cnblogs.com/page/188170/>

## Nginx 的配置 nginx.conf

/etc/hosts 里把 www.xtuer.com 映射到 127.0.0.1，这样可以在本地浏览器访问 <http://www.xtuer.com>

> Mac 下 MAMP 的 nginx 的配置文件路径为 /Applications/MAMP/conf/nginx/nginx.conf

```js
user                        root admin;
worker_processes            2;

events {
    worker_connections      1024;
}

http {
    include                 mime.types;
    default_type            text/html;
    gzip                    on;
    gzip_types              text/css text/x-component application/x-javascript application/javascript text/javascript text/x-js text/richtext image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
    sendfile                on;

    upstream app_server {
        server 127.0.0.1:8080 weight=4;
        server 127.0.0.1:8081 weight=2;
        server 127.0.0.1:8082 weight=1;
    }

    server {
        listen      80;
        charset     utf-8;
        server_name www.xtuer.com; # host_name of URL
        
        location / {
            proxy_redirect   off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://app_server;
        }
        
        # static resources: jpg, png, js, css, etc.
        location ~ \.(gif|jpg|jpeg|png|bmp|swf|js|css|html)$ {
            root /Applications/MAMP/htdocs/www.xtuer.com;
            expires 30d;
        }
        
        # url rewrite, better to be included in separated file
        rewrite /image/cm.png /image/Countryman.png last;
        rewrite /image/png/(.*) /image/$1.png last;
    }
}
```

访问 <http://www.xtuer.com/**> Nginx 会匹配 URL，如果是静态图片，js, css 等资源，则从 Nginx 所在电脑访问，其他的则根据 `weight` 值分发给 `upstream` 里面的 `AppServer` 处理，某个 AppServer 不能连接也不会影响系统的访问。

## 同一台电脑启动多个 AppServer 组成`集群`

##### 以 Maven module `Xbox` 为例：

1. 修改 pom.xml 里的 port 为不同的端口号  

   ```js
   <plugin>
       <!--嵌入式的 Tomcat Web Server-->
       <groupId>org.apache.tomcat.maven</groupId>
       <artifactId>tomcat7-maven-plugin</artifactId>
       <version>${tomcat.version}</version>
       <configuration>
           <port>8081</port>
           <path>/</path> <!--Content Path用 /，而不是项目名-->
           <uriEncoding>UTF-8</uriEncoding> <!--处理 GET 请求的编码-->
       </configuration>
   </plugin>
   ```

2. 执行 `mvn tomcat7:run` 启动 Xbox

3. 重复 1, 2 多次，启动多个 Xbox，但是监听不同的端口号

4. 配置 nginx 的 upstream，让多个 Xbox 组成集群

   ```js
   upstream app_server {
       server 127.0.0.1:8080 weight=4;
       server 127.0.0.1:8081 weight=2;
       server 127.0.0.1:8082 weight=1;
   }
   ```

5. 浏览器里多次访问如 <http://www.xtuer.com/error>，就可以看到不同的 AppServer 处理请求的次数因为 weight 的不同而不同。

> Gradle 的 gretty 插件目前还不支持多实例运行。

## Mac 下用 MAMP 安装的 Nginx 的命令

**重新加载配置前测试配置语法的合法性很重要:**

1. 测试配置文件语法：`sudo /Applications/MAMP/Library/bin/nginxctl -t`
2. 重新加载配置文件：`sudo /Applications/MAMP/Library/bin/nginxctl -s reload`