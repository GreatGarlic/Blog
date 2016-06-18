---
title: 本地服务映射为外网可访问
date: 2016-06-13 11:34:40
tags: Mac
---

例如我们开发了一个网站，运行在我们自己的电脑上，本地访问地址是 <http://localhost.com:8080>，但是只能在自己的电脑和局域网访问，外网访问不了。如果需要外网能访问我们的网站，则需要:

* 购买一个域名和空间，把我们的网站部署上去
* 使用工具把本地的网站服务映射为外网可访问的，例如 [Ngrok](https://ngrok.com)，可支持 Mac，Windows，Linux

<!--more-->

## Ngrok 的使用
1. 安装 Ngrok: 到 [Ngrok](https://ngrok.com) 下载，解压即可
2. 运行 Ngrok: `./ngrok http 8080`，输出

    ```
    Version                       1.7/1.7                                                                                                                                             
    Forwarding                    http://xtuer.ngrok.cc -> 127.0.0.1:8080                                                                                                             
    Forwarding                    https://xtuer.ngrok.cc -> 127.0.0.1:8080                                                                                                            
    Web Interface                 127.0.0.1:4040                                                                                                                                      
    # Conn                        0                                                                                                                                                   
    Avg Conn Time                 0.00ms
    ```

3. 使用 `8080` 端口启动我们的电脑上的网站项目 (因为上面启动 Ngrok 时使用的是 8080)
4. 从浏览器中访问 <http://3f506cbe.ngrok.io>，发现访问的是我们的网站
5. 外网访问: 使用手机的 4G 网络，在手机的浏览器中访问 <http://3f506cbe.ngrok.io>，访问的也是我们的网站

> Ngrok 的服务器在国外，速度有些慢，国内有 ngrok.cc 对其封装了一下，速度快很多  
> 同样的工具，有如`花生壳`

## Ngrok.cc 的使用
1. 访问 <http://www.ngrok.cc/login> 注册
2. 登陆，然后到域名列表里添加新的域名，例如 `xtuer` (不带 com，因为是作为 ngrok.cc 的子域名使用)
3. 基本信息里有一个 token，是 ngrok.cc 给我们的授权码，一会配置文件里需要使用
4. 访问 <http://www.ngrok.cc>，下载对应系统的 Ngrok
5. 解压 Ngrok，修改配置文件 `ngrok.cfg`
    * `auth_token` 为 3 中提到的 `token`
    * `subdomain` 为 2 中添加的子域名如 `xtuer`
    * `proto` 下面的 `http` 的端口号为本地 Web 服务的端口，例如 `8080`
6. 启动 Ngrok: `./ngrok -config ngrok.cfg start sunny`，输出

    ```                                                                                                                                                                                      
    Tunnel Status                 online                                                                                                                                              
    Version                       1.7/1.7                                                                                                                                             
    Forwarding                    http://xtuer.ngrok.cc -> 127.0.0.1:8080                                                                                                             
    Forwarding                    https://xtuer.ngrok.cc -> 127.0.0.1:8080                                                                                                            
    Web Interface                 127.0.0.1:4040                                                                                                                                      
    # Conn                        0                                                                                                                                                   
    Avg Conn Time                 0.00ms                         
    ```

7. 启动本地的 Web 服务 (例如使用 8080 端口)
8. 访问 <http://xtuer.ngrok.cc>，就访问了我们本地的 Web 服务




