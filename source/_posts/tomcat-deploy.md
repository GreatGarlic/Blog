---
title: Tomcat 部署
date: 2016-10-14 17:24:29
tags: Java
---
部署工程为 Tomcat 的默认工程，工程的 war 包为 **web-mix.jar**

1. 创建目录 **/Users/Biao/Desktop/data**
2. 复制 **web-mix.jar** 到目录 **/Users/Biao/Desktop/data**
3. 解压 **web-mix.jar** 得到目录结构 (Tomcat 启动后不会自动解压)  

    ```
    data
    ├── web-mix
    │   ├── META-INF
    │   │   └── MANIFEST.MF
    │   └── WEB-INF
    │       ├── asset
    │       ├── ......
    ```

    > 解压命令: `rm -rf web-mix; unzip web-mix.war -d web-mix`

4. 在 `<tomcat>/conf/Catalina/localhost` 下创建文件 **ROOT.xml**:

    ```xml
    <Context path="/" docBase="/Users/Biao/Desktop/data/web-mix"
        debug="0" privileged="true" reloadable="false">
    </Context>
    ```

    > **docBase**: 工程所在路径  
    > **path**: 工程的 context path
5. 启动 Tomcat
6. 访问 <http://localhost:8080> 显示的是上面工程的页面，而不是 Tomcat 默认的主页

    > 不用删除 `<tomcat>/webapps/ROOT`，继续放在那里就可以了
