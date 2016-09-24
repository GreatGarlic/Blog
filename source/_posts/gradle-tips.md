---
title: Gradle Tips
date: 2016-04-05 17:30:45
tags: Gradle
---

## 依赖的 Scope
* compile
* runtime
* compileOnly (新版才支持)
* testCompile
* testRuntime

<!-- more -->   

## 项目依赖关系

```
gradle dep
gradle dependencies
```

## 依赖其他子项目
```
compile project(':dependency-project')
```

## 打包工程

```
gradle assemble // 编译，打包
gradle build // 编译，执行测试，打包
```

## 定制包名
打包时制定包名，例如默认打包出来是 `Exam.war`，而我们需要的是 `Exam.zip`

```
war.archiveName 'Exam.zip'
```

## 编译工程

```
gradle compileJava
```

## 指定编译的输出目录

```
buildDir = new File(rootProject.projectDir, '../build/' + project.name)
```

## 指定 Maven 仓库

```groovy
allprojects {
    repositories {
        mavenLocal()
        mavenCentral()
        maven{ url 'http://maven.edu-edu.com.cn/content/groups/public/' }
        
        // 带认证的库
        maven {
            url 'http://maven.edu-edu.com.cn/content/groups/public/'
            credentials {
                username 'admin'
                password 'admin123'
            }
        }
    }
}
```

## 上传 jar 包到 Maven 仓库
* 命令: gradle uploadArchives
* Release 版本地址: http://maven.edu-edu.com.cn/nexus/content/repositories/releases/
* Snapshop 版本地址: http://maven.edu-edu.com.cn/nexus/content/repositories/snapshots/
* 上传的地址和下载的地址是有区别的

```groovy
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "http://maven.edu-edu.com.cn/nexus/content/repositories/releases/") {
                authentication(userName: "admin", password: "admin123")
            }
            
            // 以下三项可选
            pom.groupId    = "$project.group"
            pom.artifactId = "$artifactId" // 默认为项目的目录名
            pom.version    = "$project.version"
        }
    }
}
```

## 打包时过滤掉不需要的 jar 包

```groovy
war {    
    // 打包时只包含以下几个 jar 包
    classpath = classpath.filter { file ->
        file.name.startsWith('lms-client') ||
        file.name.startsWith('dict-client') ||
        file.name.startsWith('e-platform-client')
    }
}
```

## 打包时替换 `META-INF/context.xml` 中的占位符

```groovy
war {
    // gradle build 打包时替换 META-INF 下的文件
    from("$webAppDir/META-INF") {
        eachFile {
            it.filter(ReplaceTokens, tokens: loadConfiguration())
        }
        into('META-INF')
    }
}
```

## 运行时替换 `META-INF/context.xml` 中的占位符
任务 `prepareInplaceWebApp` 执行完后执行自定义任务，例如替换 `context.xml` 中的占位符

```groovy
project.afterEvaluate {
    // prepareInplaceWebApp 的时候替换 META-INF 下的文件，例如生成 context.xml
    tasks.getByPath('prepareInplaceWebApp').doLast {
        copy {
            from("$webAppDir/META-INF") {
                eachFile {
                    it.filter(ReplaceTokens, tokens: loadConfiguration())
                }
            }
            into("$buildDir/inplaceWebapp/META-INF")
        }
    }
}
    ```

## 获取命令中的参数
推荐使用 -D 的方式

```groovy
命令: gradle -Denv=development clean assemble
获取: def environment = System.getProperty('env', 'development')

命令: gradle -Penv=develop clean assemble
获取: def environment = hasProperty('env') ? env : 'development'
```

## 资源文件内容动态替换

```groovy
def loadConfiguration() {
    // 获取 gradle 参数中 -Penv 的值: gradle -Penv=development clean assemble
    def environment = hasProperty('env') ? env : 'development'
    def configFile = file('config.groovy') // 配置文件
    return new ConfigSlurper(environment).parse(configFile.toURI().toURL()).toProperties()
}
    
processResources {
    // src/main/resources 下的文件中 @key@ 的内容使用 config.groovy 里对应的进行替换
    from(sourceSets.main.resources.srcDirs) {
        filter(ReplaceTokens, tokens: loadConfiguration())
    }
}
```

```groovy
// config.groovy 的内容
environments {
    development {
        db {
            host = '127.0.0.1'
            username = 'root'
            password = 'root'
            database = 'db_exam'
        }
    }
    
    production {
        db {
            host = '192.168.10.123'
            username = 'root'
            password = 'sluper'
            database = 'db_exam'
        }
    }
}
```

## 使用嵌入式服务器 Tomcat

```groovy
apply plugin: 'org.akhikhl.gretty'
    
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'org.akhikhl.gretty:gretty:1.4.0'
    }
}
    
gretty {
    port = 8080
    contextPath = ''
    servletContainer = 'tomcat7'
}
```

```groovy
// gretty 的更多配置
gretty {
    httpPort    = 8080
    debugPort   = httpPort + 1
    servicePort = httpPort + 2
    statusPort  = httpPort + 3
    httpsPort   = httpPort + 4
    httpsEnabled = true
    contextPath  = ''
    jvmArgs = ['-Xmx1024M', '-XX:PermSize=128M', '-XX:MaxPermSize=256M']
    servletContainer = 'jetty7'
    scanInterval = 0
    inplaceMode  = 'hard'
    debugSuspend = false
    enableNaming = true // 启用 JNDI
    managedClassReload = true
}
```
> Tomcat 的 GET 请求的默认编码是 ISO8859-1，如果 GET 需要使用 UTF-8 的话，需要在 `server.xml` 中设置 `URIEncoding` 为 UTF-8:
> 
```
<Connector port="8080" protocol="HTTP/1.1"
         connectionTimeout="20000"
         redirectPort="8443"
         URIEncoding="UTF-8"/>
```
> 在 Gretty 中没有提供设置 `URIEncoding` 的选项，但是可以设置 `serverConfigFile` 引用 Tomcat 的 `server.xml` 来达到目的，例如:
> 
```
gretty {
    httpPort = 8080
    contextPath = ''
    servletContainer = 'tomcat7'
    serverConfigFile = "server.xml"

    inplaceMode = 'hard'
    debugSuspend = false
    managedClassReload      = true
    recompileOnSourceChange = false
}
```

## Gretty 热部署优化

> Gretty 热部署的相关配置，下面几项都默认设置为 true (reload 表示重新加载整个 web 应用):
>
> * `recompileOnSourceChange` // Java 文件变化后自动编译
> * `reloadOnClassChange` // Class 文件变化后 reload
> * `reloadOnConfigChange` // 配置文件如 web.xml 变化后 reload
> * `reloadOnLibChange` // lib 目录下的 jar 变化后 reload
> 
> 例如 Java 文件修改保存后会自动编译，导致 Class 文件变化，然后 gretty 会重新加载整个 Web 应用，例如应用启动时要执行计划任务，检查数据库的结构等，这样的热部署效率太低。
>
> 如果 Class 文件变化后，能不能只加载 Class 文件，而不是重新加载整个 Web 应用呢？设置 `managedClassReload=true` 即可 (使用的是 SpringLoaded 来进行热加载，并自动设置 reloadOnClassChange=false):
    
```groovy
gretty {
    port = 80
    contextPath = '/'
    servletContainer = 'tomcat7'
    
    managedClassReload      = true
    recompileOnSourceChange = false // 禁止自动编译过度频繁的进行热部署
}
```

## 多模块工程
* `parent` 是父模块
* `mix1` 和 `mix2` 是子模块

```
└── parent
    ├── mix1
    │   ├── build.gradle
    │   └── src
    │       └── main
    │           ├── java
    │           └── resources
    ├── mix2
    │   ├── build.gradle
    │   └── src
    │       └── main
    │           ├── java
    │           └── resources
    ├── build.gradle
    └── settings.gradle
```

```groovy
// parent/settings.gradle
include 'mix1'
include 'mix2'
```

```groovy
// parent/build.gradle
// 所有模块生效
allprojects {
    ext {
        // gradle build or gradle -Dprofile=product build
        profile = System.getProperty("profile", "dev")
    }
    
    println '++ Building: ' + project.name
}
    
// 某一个指定模块生效
project('mix1') {
    println 'Building module mix1'
}

// 多个匹配的子模块生效
configure(subprojects.findAll {
        it.name.equals('mix1') ||
        it.name.equals('mix2')
}) {
    apply plugin: 'java'
}
```

进入 parent 目录:   
* 执行 `gradle build` 构建所有的模块
* 执行 `gradle :mix1:build` 构建子模块 `mix1`

---

## 获取路径

```
println "${rootProject.projectDir}/config"
    
不能用
println ${rootProject.projectDir} + "/config"
```

## settings.gradle

除了构建脚本文件，Gradle 还定义了一个约定名称的设置文件(默认为settings.gradle)，该文件在初始化阶段被执行，对于多项目构建必须保证在根目录下有 settings.gradle 文件，对于单项目构建设置文件是可选的

## 强制 Gradle 使用 UTF-8

```
打开 gradle/bin 目录下的 gradle.bat 文件，修改 12 行附近的代码为
set DEFAULT_JVM_OPTS="-Dfile.encoding=UTF-8"
```

或者

```
gradle.properties 里添加
systemProp.file.encoding=UTF-8
```

## 创建工程目录结构
Gradle 默认没有提供创建各种项目目录结构的任务，但是可以使用 `gradle-templates` 来创建工程的目录结构。

```groovy
apply plugin: 'java'
apply plugin: 'templates'

repositories {
    mavenCentral()
}

buildscript {
    repositories {
        maven {
            url 'http://dl.bintray.com/cjstehno/public'
        }
    }

    dependencies {
        classpath 'gradle-templates:gradle-templates:1.5'
    }
}
```

运行 `gradle tasks` 可以看到有很多创建不同项目结构的任务

* 使用 `gradle initJavaProject` 创建 Java 应用程序的目录结构
* 使用 `gradle initWebappProject` 创建 Web 项目的目录结构

## 使用 Spring 的应用程序打包
Gradle 默认的打包任务 `jar` 不能带上依赖的类，在有依赖 Spring 的项目中，最好是使用 `shadowJar` 来打包，其会合并 spring 冲突的配置文件，例如 fatJar 则不会。

> 如果是可执行的 jar 包，配置 mainClassName 为 jar 包启动运行的类，否则可以忽略掉。

```groovy
apply plugin: 'java'
apply plugin: 'application'
apply plugin: 'com.github.johnrengelman.shadow'

buildscript {
    repositories { jcenter() }
    dependencies { classpath 'com.github.jengelman.gradle.plugins:shadow:1.2.3' }
}

mainClassName = 'Foo'

jar {
    manifest { attributes 'Main-Class': mainClassName }
}

// 打包命令: gradle clean shadowJar
shadowJar {
    mergeServiceFiles('META-INF/spring.*')
}
```



## 参考
* [Gradle 替代 Maven](http://my.oschina.net/enyo/blog/369843)
* [Gretty Hot deployment](http://akhikhl.github.io/gretty-doc/Hot-deployment.html)
* [Gradle脚本基础全攻略](http://blog.csdn.net/yanbober/article/details/49314255)
