---
title: Gradle Deploy
date: 2017-08-01 13:35:01
tags: Gradle
---

项目打包后一般可以按照以下几个步骤进行部署：

1. 把 war 包上传到服务器（可以使用 FTP）
2. 停止 tomcat: `<tomcat>/bin/shutdown.sh`
3. 删除 tomcat 下项目的文件: `rm -rf <project_path>`
4. 解压 war 包到项目的路径: `unzip project.war -d <project_path>`
5. 启动 tomcat: `<tomcat>/bin/startup.sh`
6. 删除上传的 war 包

每次部署都要重复这么多个步骤，效率不高，为了提高效率，借助 [Gradle 的 deploy 插件](https://gradle-ssh-plugin.github.io)，一条命令就完成上面的这些事了。<!--more-->

## build.gradle

下面是 gradle deploy 的关键配置，根据项目具体情况进行修改，然后把它加到项目的 build.gradle:

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'org.hidetake:gradle-ssh-plugin:2.9.0'
    }
}

apply plugin: 'org.hidetake.ssh'

remotes {
    webServer {
        host = '192.168.82.130'
        user = 'root'
        identity = file('/Users/Biao/.ssh/id_rsa')
    }
}

task deploy {
    doLast {
        ssh.run {
            session(remotes.webServer) {
                put from: '/project/build/libs/mini.war', into: '/root'
                execute """
                    source /root/.bash_profile;
                    /usr/local/tomcat/bin/shutdown.sh;
                    rm -rf /usr/local/tomcat/webapps/mini;
                    unzip /root/mini.war -d /usr/local/tomcat/webapps/mini;
                    /usr/local/tomcat/bin/startup.sh;
                    rm -rf /root/mini.war
                """
            }
        }
    }
}
```

| 参数       | 说明                                       |
| -------- | ---------------------------------------- |
| host     | 服务器的 IP 或者域名                             |
| user     | ssh 使用的用户名                               |
| identity | ssl 的 key 文件 (首先在服务器上把 id_rsa.pub 加入到授权文件中) |
| from     | 要上传的文件路径                                 |
| into     | 上传到服务器的目录                                |
| execute  | 上传完成后在服务器上接着执行的命令，使用分号分割不同的命令            |

> execute 中先执行 `source /root/.bash_profile` 是为了使环境变量生效，否则就不能调用 java 命令启动 tomcat。
>
> **.bash_profile** 中设置了 java 的环境变量:
>
```
export JAVA_HOME="/usr/local/jdk1.8.0_144"
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export PATH
```

## 部署

命令行下进入项目文件夹，执行命令 `gradle deploy` 即可。