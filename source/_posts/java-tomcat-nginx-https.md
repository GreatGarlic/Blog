---
title: Nginx + Tomcat 使用 Https
date: 2017-07-26 10:27:07
tags: Java
---

Nginx 作为前端反向代理或者负载均衡，Tomcat 不需要自己处理 https，**https 由 Nginx 处理**:

* 用户首先和 Nginx 建立连接，完成 SSL 握手
* 而后 Nginx 作为代理以 http 协议将请求转发给 Tomcat 处理
* Nginx 再把 Tomcat 的输出通过 SSL 加密发回给用户

这中间是透明的，Tomcat 只是在处理 http 请求而已（默认监听 8080 端口）。因此，这种情况下不需要配置 Tomcat 的 SSL，只需要配置 Nginx 的 SSL，Tomcat 和 Nginx 需要配置以下几项:

* Nginx 中启用 https:

  ```nginx
  http {
      include      mime.types;
      default_type text/html;
      gzip         on;
      gzip_types   text/css text/x-component application/x-javascript application/javascript text/javascript text/x-js text/richtext image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
      sendfile     on;

      # Tomcat 服务器集群
      upstream app_server {
          server 127.0.0.1:8080 weight=4;
          server 127.0.0.1:8081 weight=2;
          server 127.0.0.1:8082 weight=1;
      }

      server {
          listen      443; # https 的默认端口是 443
          charset     utf-8;
          server_name www.xtuer.com; # host_name of URL

          # 启用 https
          ssl on;
          ssl_certificate     /Users/Biao/Desktop/cert/server.crt;
          ssl_certificate_key /Users/Biao/Desktop/cert/server.key;

          location / {
              proxy_redirect   off;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

              # 把 https 的协议告知 Tomcat，否则 Tomcat 可能认为是 http 的请求
              proxy_set_header X-Forwarded-Proto $scheme;
              # 请求转发给 Tomcat 集群处理
              proxy_pass http://app_server;
          }
      }
  }
  ```
  > 关键是以下几项:
  >
  > * ssl on
  > * ssl_certificate
  > * ssl_certificate_key 
  > * X-Forwarded-Proto

* Tomcat 的 server.xml 的 Host 中配置 Valve:

  ```xml
  <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
      <Valve className="org.apache.catalina.valves.RemoteIpValve" remoteIpHeader="X-Forwarded-For" protocolHeader="X-Forwarded-Proto" protocolHeaderHttpsValue="https"/>
  </Host>
  ```

  > * **X-Forwarded-Proto** 是为了正确地识别实际用户发出的协议是 http 还是 https。
  > * **X-Forwarded-For** 是为了获得实际用户的 IP。

<!--more-->

## 自签证书

https 需要的证书可以去购买，也可以申请免费的证书，不过我们也可以自签一个证书在本地进行测试（自签的证书在访问时需要用户手动确认信任证书才能继续访问），步骤如下:

1. openssl genrsa -des3 -out server.key 1024

   ```
   Generating RSA private key, 1024 bit long modulus
   .....++++++
   ..++++++
   e is 65537 (0x10001)
   Enter pass phrase for server.key:              # 输入密码 123456
   Verifying - Enter pass phrase for server.key:  # 输入密码 123456
   ```

2. openssl req -new -key server.key -out server.csr

   ```
   Enter pass phrase for server.key:             # 输入上面设置的密码 123456
   You are about to be asked to enter information that will be incorporated
   into your certificate request.
   What you are about to enter is what is called a Distinguished Name or a DN.
   There are quite a few fields but you can leave some blank
   For some fields there will be a default value,
   If you enter '.', the field will be left blank.
   -----
   Country Name (2 letter code) [AU]:CN                    # 国家的代码，中国输入 CN 即可
   State or Province Name (full name) [Some-State]:Beijing # 省
   Locality Name (eg, city) []:Beijing                     # 城市
   Organization Name (eg, company) [Internet Widgits Pty Ltd]:xo # 公司，随意
   Organizational Unit Name (eg, section) []:xo                  # 部门，随意
   Common Name (e.g. server FQDN or YOUR name) []:xtuer.com      # 可以随意输入，不一定要是域名
   Email Address []:            # 回车，不需要输入

   Please enter the following 'extra' attributes
   to be sent with your certificate request
   A challenge password []:     # 回车，不需要输入
   An optional company name []: # 回车，不需要输入
   ```

3. cp server.key server.key.org

4. openssl rsa -in server.key.org -out server.key

   ```
   Enter pass phrase for server.key.org: # 输入上面设置的密码 123456
   writing RSA key
   ```

5. openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

   ```
   Signature ok
   subject=/C=CN/ST=Beijing/L=Beijing/O=xo/OU=xo/CN=xtuer.com
   Getting Private key
   ```

6. 把生成的 server.crt 和 server.key 配置到 Nginx

   ```
   # [1]
   ssl on;
   ssl_certificate     /Users/Biao/Desktop/cert/server.crt;
   ssl_certificate_key /Users/Biao/Desktop/cert/server.key;

   # [2] 把 https 的协议告知 Tomcat，否则 Tomcat 可能认为是 http 的请求
   proxy_set_header X-Forwarded-Proto $scheme; 
   ```

7. 加载 Nginx 配置(以 Mac 下 MAMP 为例)

   * 测试配置文件语法：`sudo /Applications/MAMP/Library/bin/nginxctl -t`
   * 重新加载配置文件：`sudo /Applications/MAMP/Library/bin/nginxctl -s reload`

8. 使用 https 访问网站，例如 <https://www.xtuer.com>

9. 浏览器会提示证书不可信任，选择信任即可

   ```
   Your connection is not private

   Attackers might be trying to steal your information from www.xtuer.com (for example, passwords, messages, or credit cards). NET::ERR_CERT_AUTHORITY_INVALID
   ```

## ​参考资料

* [Nginx + Tomcat + SSL 识别 https 还是 http](http://blog.csdn.net/woshizhangliang999/article/details/51861998)
* [Https 无法与服务器建立安全连接](http://blog.csdn.net/ws_zll/article/details/8486033)