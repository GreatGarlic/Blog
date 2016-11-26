---
title: Gradle 修改 Maven 仓库
date: 2016-11-22 19:32:51
tags: Gradle
---
Gradle 的默认仓库在国内下载太慢了，可以切换到国内的 Maven 镜像仓库，如阿里的 Maven 库，又或者是换成自建的 Maven 私服。

一个简单的办法，修改项目的 build.gradle，将 jcenter() 或者 mavenCentral() 替换掉即可:

```xml
allprojects {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    }
}
```

<!--more-->

但是如果项目太多，一个一个的修改工作量就太大了，可以通过修改 Gradle 使用的默认 Maven 仓库，这样就不需要单独的修改每个项目了，将下面这段配置复制到 `<USER_HOME>/.gradle/init.gradle` 即可:

```xml
allprojects {
    repositories {
        def REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public/'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository) {
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com/')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $REPOSITORY_URL."
                    remove repo
                }
            }
        }
        maven {
            url REPOSITORY_URL
        }
    }
}
```

来源于 <https://yrom.net/blog/2015/02/07/change-gradle-maven-repo-url/>
