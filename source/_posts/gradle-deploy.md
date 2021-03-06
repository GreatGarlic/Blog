---
title: Gradle Deploy
date: 2017-08-01 13:35:01
tags: [Gradle, SpringWeb]
---

项目打包后一般可以按照以下几个步骤进行部署：

1. 选择正确的环境打包: 测试环境、线上环境等
2. 把 war 包上传到服务器（使用 FTP、scp 等）
3. 停止 tomcat: `<tomcat>/bin/shutdown.sh`
4. 删除服务器上的项目文件: `rm -rf <project_path>`
5. 解压 war 包到项目路径下: `unzip project.war -d <project_path>`
6. 启动 tomcat: `<tomcat>/bin/startup.sh`
7. 删除上传的 war 包
8. 如果有 N 个服务器，就需要重复 2 到 7 共 N 次

每次部署都要重复这么多步骤，效率不高，而且容易疏忽出错，为了解决这些问题，借助 [Gradle 的 deploy 插件](https://gradle-ssh-plugin.github.io)，一条命令 `gradle deploy` 就完成上面的这些事了。<!--more-->

## build.gradle

下面是 gradle deploy 的关键配置，请根据项目具体情况进行修改:

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
        password = 'xxxx'
    }
}

task deploy {
    doLast {
        ssh.settings {
            knownHosts = allowAnyHosts
        }
      
        ssh.run {
            session(remotes.webServer) {
                put from: "${buildDir}/libs/mini.war", into: '/root'
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
| user     | 服务器的用户名                                  |
| password | 服务器的用户的密码                                |
| identity | ssl 的签名文件 (首先在服务器上把 id_rsa.pub 加入到授权文件中)，使用它的时候就不需要使用密码了 |
| from     | 本地要上传的文件路径<br>工程路径: ${projectDir}<br>构建路径: ${buildDir} |
| into     | 上传到服务器的目录                                |
| execute  | 上传完成后在服务器上接着执行的命令，可以执行多条命令，使用分号分割不同的命令   |

## 项目部署

* 把上面的内容添加到 **build.gradle**，执行命令 `gradle deploy` 即可
* 把上面的内容保存到 **deploy.gradle**，执行命令 `gradle -b deploy.gradle deploy` 即可

## 常见问题

### 1) 怎么同时部署到多台服务器?

在 remotes 中定义多个 server 的信息，然后在 ssh.run 下面创建多个 session，每个 session 中使用对应的 server 配置。

### 2) 不能使用用户名密码访问

服务器端为了安全，不允许使用用户名密码访问，而是使用 ssl 的签名文件进行访问，则在 server 配置中使用 identity 指定签名文件即可:

```
remotes {
    webServer {
        host = '192.168.82.130'
        user = 'root'
        identity = file('/Users/Biao/.ssh/id_rsa')
    }
}
```

### 3) 报错 reject HostKey ...

服务器 IP 没有加入到 .ssh/known_hosts 文件，执行 ssh user@host，确认把服务器 IP 加入到 known_hosts 即可。

也可以偷懒，在 ssh.settings 中设置 knownHosts = allowAnyHosts，这样就不需要执行 ssh 手动把服务器 IP 加入到 known_hosts。

### 4) 停止或启动 tomcat 报错

发现是不能使用 java 命令，原因是手动安装 JDK 并在 .bash_profile 中设置好了 JAVA_HOME 和 java 加入到了 $PATH:

```
export JAVA_HOME="/usr/local/jdk1.8.0_144"
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export PATH
```

但是远程执行时是不知道 .bash_profile 里的配置的，需要在 execute 中先执行一下 source /root/.bash_profile。

如果创建了 JDK/bin/java 的软连接到 /usr/bin/java 的话，就不需要先执行 source /root/.bash_profile 了:

```
/usr/bin/java -> /usr/local/edu/jdk/bin/java
```

## 参考资料

上面的简单介绍，大多数时候已经够用了，想了解更详细的内容请参考 [Gradle 的 deploy 插件](https://gradle-ssh-plugin.github.io)。

熟悉上面的基础后，同时也了解 gradle 的语法，也可以 build 和部署的任务写在一起，或者使用包含文件的方式，下面附上一个实际项目中的 build.gradle 作为参考，**执行 gradle deploy 就能进行重新编译、打包、发布:**

```groovy
plugins {
    id 'war'
    id 'java'
    id 'org.hidetake.ssh'   version '2.9.0'
    id 'org.akhikhl.gretty' version '2.0.0'
}

gretty {
    httpPort = 8080
    contextPath = ''
    servletContainer = 'tomcat8'

    inplaceMode  = 'hard'
    debugSuspend = false
    managedClassReload      = true
    recompileOnSourceChange = true

    // 升级 gretty 自带的 springloaded
    jvmArgs = ["-javaagent:${project.projectDir}/springloaded-1.2.8.RELEASE.jar", '-noverify']
}

////////////////////////////////////////////////////////////////////////////////
//                                   Maven 依赖                               //
////////////////////////////////////////////////////////////////////////////////
repositories {
    mavenCentral()
}

ext {
    // 运行和打包的环境选择, 默认是开发环境
    // 构建: gradle clean build
    //       gradle clean build -Denv=production
    // 部署: gradle clean deploy
    //       gradle clean deploy -Denv=production
    environment = System.getProperty("env", "development") // 获取 gradle 参数中 env 的值，选择环境
    war.archiveName = 'Fox.zip' // 打包的文件名，不用 war 包自动解压的话，用 .zip 会更好一些
}

ext.versions = [
    spring        : '5.0.2.RELEASE',
    springSecurity: '5.0.0.RELEASE',
    springSession : '1.3.1.RELEASE',
    servlet       : '4.0.0',
    lombok        : '1.16.18',
    fastjson      : '1.2.41',
    thymeleaf     : '3.0.9.RELEASE',
    mysql         : '5.1.21',
    mybatis       : '3.4.5',
    mybatisSpring : '1.3.1',
    druid         : '1.1.5',
    validator     : '6.0.5.Final',
    commonsLang   : '3.7',
    commonsFileupload: '1.3.3',
    snakeyaml     : '1.19',
    easyOkHttp    : '1.1.3',
    logback       : '1.2.3',
    junit         : '4.12',
    jclOverSlf4j  : '1.7.25'
]

dependencies {
    compile(
            "org.springframework:spring-webmvc:${versions.spring}",
            "org.springframework:spring-context-support:${versions.spring}",
            "org.springframework.security:spring-security-web:${versions.springSecurity}",
            "org.springframework.security:spring-security-config:${versions.springSecurity}",
            "org.springframework.session:spring-session-data-redis:${versions.springSession}",
            "com.alibaba:fastjson:${versions.fastjson}",
            "org.thymeleaf:thymeleaf-spring5:${versions.thymeleaf}",
            "mysql:mysql-connector-java:${versions.mysql}",
            "org.springframework:spring-jdbc:${versions.spring}",
            "org.mybatis:mybatis-spring:${versions.mybatisSpring}",
            "org.mybatis:mybatis:${versions.mybatis}",
            "com.alibaba:druid:${versions.druid}",
            "org.hibernate.validator:hibernate-validator:${versions.validator}",
            "org.apache.commons:commons-lang3:${versions.commonsLang}",
            "commons-fileupload:commons-fileupload:${versions.commonsFileupload}",
            "org.yaml:snakeyaml:${versions.snakeyaml}",
            "com.mzlion:easy-okhttp:${versions.easyOkHttp}",
            "ch.qos.logback:logback-classic:${versions.logback}",
            "org.slf4j:jcl-over-slf4j:${versions.jclOverSlf4j}"
    )

    compileOnly("org.projectlombok:lombok:${versions.lombok}")
    compileOnly("javax.servlet:javax.servlet-api:${versions.servlet}")
    testCompile("org.springframework:spring-test:${versions.spring}")
    testCompile("junit:junit:${versions.junit}")
}

////////////////////////////////////////////////////////////////////////////////
//                                  资源动态替换                                //
////////////////////////////////////////////////////////////////////////////////
processResources {
    // src/main/resources 下的文件中 @key@ 的内容使用 config.groovy 里对应的进行替换
    println "==> Load configuration for '${environment}'"
    def configFile = file('config.groovy') // 配置文件
    def props = new ConfigSlurper(environment).parse(configFile.toURI().toURL()).toProperties()

    from(sourceSets.main.resources.srcDirs) {
        filter(org.apache.tools.ant.filters.ReplaceTokens, tokens: props)
    }
}

////////////////////////////////////////////////////////////////////////////////
//                                   项目部署                                  //
////////////////////////////////////////////////////////////////////////////////
remotes {
    server {
        host = '192.168.82.133'
        user = 'root'
        // password 和 identity 只用其中一个
        // password = 'xxx'
        identity = file("${System.properties['user.home']}/.ssh/id_rsa")
    }
}

ssh.settings {
    knownHosts = allowAnyHosts
}

task deploy(dependsOn: war) {
    def targetDir = '/data/xtuer.com'
    doLast {
        ssh.run {
            session(remotes.server) {
                put from: "${buildDir}/libs/${war.archiveName}", into: "${targetDir}"
                execute """
                    source /root/.bash_profile;
                    /usr/local/tomcat/bin/shutdown.sh;
                    rm -rf ${targetDir}/ROOT;
                    unzip -u ${targetDir}/${war.archiveName} -d ${targetDir}/ROOT > /dev/null;
                    kill `ps aux | grep -i tomcat | grep -v grep | awk '{print \$2}'`;
                    /usr/local/tomcat/bin/startup.sh;
                    rm -rf ${targetDir}/${war.archiveName};
                """
            }
        }
    }
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

> 可以把部署的部分放到 deploy.gradle 文件，然后 build.gradle 中 apply from: 'deploy' 的方式。

