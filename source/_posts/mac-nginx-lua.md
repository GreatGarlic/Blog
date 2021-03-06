---
title: Nginx 安装 Lua 支持
date: 2017-10-14 20:21:30
tags: Mac
---

Nginx 支持 Lua 需要安装 lua-nginx-module 模块，一般常用有 2 种方法: 

* 编译 Nginx 的时候带上 lua-nginx-module 模块一起编译

* 使用 OpenResty: Nginx + 一些模块，默认启用了 Lua 支持(推荐使用此方式)

  > [OpenResty](https://openresty.org/cn/openresty.html) is just an enhanced version of [Nginx](https://openresty.org/cn/nginx.html) by means of addon modules anyway. You can take advantage of all the exisitng goodies in the [Nginx](https://openresty.org/cn/nginx.html) world.
  >
  > ​
  >
  > OpenResty® 是一个基于 [Nginx](https://openresty.org/cn/nginx.html) 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。
  >
  > OpenResty® 通过汇聚各种设计精良的 [Nginx](https://openresty.org/cn/nginx.html) 模块（主要由 OpenResty 团队自主开发），从而将 [Nginx](https://openresty.org/cn/nginx.html) 有效地变成一个强大的通用 Web 应用平台。这样，Web 开发人员和系统工程师可以使用 Lua 脚本语言调动 [Nginx](https://openresty.org/cn/nginx.html) 支持的各种 C 以及 Lua 模块，快速构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。
  >
  > OpenResty® 的目标是让你的Web服务直接跑在 [Nginx](https://openresty.org/cn/nginx.html) 服务内部，充分利用 [Nginx](https://openresty.org/cn/nginx.html) 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求,甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。<!--more-->

## OpenResty

OpenResty 的安装很方便，Mac 里使用 brew 安装，对于一些常见的 Linux 发行版本，OpenResty® 提供 [官方预编译包](https://openresty.org/cn/linux-packages.html)，CentOS 使用 yum，Ubuntu 使用 apt-get，具体请参考 <https://openresty.org/cn/installation.html>，以下以 Mac 和 CentOS 7 中安装 OpenResty 为例。

### Mac 使用 OpenResty

* 终端执行 `brew install homebrew/nginx/openresty` 把 OpenResty 安装到 **/usr/local/Cellar/openresty**

* 配置文件位于 **/usr/local/etc/openresty/nginx.conf** (可执行 `openresty -V` 从输出中找到)

* 启动 Nginx，2 种启动方式都可以

  * `sudo openresty` (openresty 其实就是 nginx 的软连接)
  * `sudo nginx` (把 /usr/local/Cellar/openresty/1.11.2.5/nginx/sbin 添加到 PATH 里，注意不同版本时的路径不同)
  * 查看是否启动了 nginx: `ps aux | grep nginx`

* 测试是否支持 Lua

  1. **/usr/local/etc/openresty/nginx.conf** 中添加

     ```
     location /lua {
         default_type 'text/html';
         content_by_lua 'ngx.say("hello world");';
     }
     ```

  2. `sudo nginx -t` 测试配置没问题，然后 `sudo nginx -s reload` 重新加载配置 (`sudo nginx -s stop` 关闭 Nginx)

  3. `curl http://localhost/lua` 输出 **hello world** 则说明 Nginx 支持 Lua

### CentOS 7 使用 OpenResty

* 终端执行下面 3 条命令把 OpenResty 安装到 **/usr/local/openresty**

  `sudo yum install yum-utils`

  `sudo yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo`

  `sudo yum install openresty`

* Nginx 的配置文件位于 **/usr/local/openresty/nginx/conf/nginx.conf** (openresty -V 中没有指定)

* 启动 Nginx，2 种启动方式都可以

  * `sudo openresty`
  * `sudo nginx`
  * 查看是否启动了 nginx: `ps -ef | grep nginx`

* 测试是否支持 Lua: 参考上面的方法

## 编译 Nginx + Lua

> 编译 Nginx 需要先准备好下面的这些工具，如果不确定是否已安装，可以在编译的时候根据出现的错误提示再进行安装
>
> * `yum install -y gcc g++ gcc-c++`
> * `yum -y install zlib zlib-devel openssl openssl--devel pcre pcre-devel`

Nginx 支持 Lua 需要依赖 LuaJIT-2.0.4.tar.gz，ngx_devel_kit，lua-nginx-module，下面介绍具体的编译过程 (都下载到 /root 目录)

1. 下载安装 **LuaJIT-2.0.4.tar.gz**

   ```
   wget -c http://luajit.org/download/LuaJIT-2.0.4.tar.gz
   tar xzvf LuaJIT-2.0.4.tar.gz
   cd LuaJIT-2.0.4
   make install PREFIX=/usr/local/luajit

   # 添加环境变量
   export LUAJIT_LIB=/usr/local/luajit/lib
   export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0
   ```

2. 下载解压 **ngx_devel_kit**

   ```
   wget https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz
   tar -xzvf v0.3.0.tar.gz
   ```

3. 下载解压 **lua-nginx-module**

   ```
   wget https://github.com/openresty/lua-nginx-module/archive/v0.10.8.tar.gz
   tar -xzvf v0.10.8.tar.gz
   ```

4. 下载安装 **nginx-1.10.3.tar.gz**

   ```
   wget http://nginx.org/download/nginx-1.10.3.tar.gz
   tar -xzvf nginx-1.10.3.tar.gz
   cd nginx-1.10.3

   # 注意ngx_devel_kit和lua-nginx-module 以实际解压路径为准
   ./configure --add-module=/root/ngx_devel_kit-0.3.0 --add-module=/root/lua-nginx-module-0.10.8

   make -j2
   make install
   ```

5. 支持 Nginx 被安装到了 **/usr/local/nginx**，配置文件为 **/usr/local/nginx/conf/nginx.conf**

6. 验证

   * 将 nginx 做成命令: `ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx`

   * **/usr/local/nginx/conf/nginx.conf** 中添加 Lua 测试代码

     ```
     location /lua {
         default_type 'text/html';
         content_by_lua 'ngx.say("hello world");';
     }
     ```

   * 启动 Nginx: `sudo nginx`

   * `curl http://localhost/lua` 输出 **hello world** 则说明 Nginx 支持 Lua

上面编译 Nginx 的内容来源于 <http://www.cnblogs.com/aoeiuv/p/6856056.html>，编译 Nginx 相对使用 OpenResty 麻烦一些，不过也不难，根据自己的喜好选择即可。