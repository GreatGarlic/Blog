---
title: Mac Homebrew
date: 2016-07-24 22:25:54
tags: Mac
---

Homebrew 可以很方便的通过终端安装许多软件，例如 Tomcat，Redis，Gradle，Tree 等，和 Ubuntu 下的 apt-get 很像，下面列出一些 Homebrew 常用命令，以安装 tomcat 为例。

<!--more-->

## 安装 Tomcat
```
brew install tomcat
```
> 安装的是最新版的版本

## 卸载 Tomcat
```
brew uninstall tomcat
```

## 更新到最新版本
```
brew upgrade tomcat
```
> 更新不会删除原来的，相当于重新安装一个最新版，也就是说配置不能共用

## 切换不同版本
```
brew switch tomcat 8.5.4
```
> 升级会出现几个不同版本的 Tomcat 同时存在，切换到指定的版本就很有必要了

## 查看已安装的 Tomcat 的版本和最新的版本信息
```
brew info tomcat
```

## 查看可安装的 Tomcat 的所有版本
```
brew search tomcat

输出:
homebrew/versions/tomcat6
homebrew/versions/tomcat7
tomcat
tomcat-native
```

## 安装指定版本的 Tomcat，例如 tomcat6
```
brew install homebrew/versions/tomcat6
```

## 列出已经安装的软件
```
brew list
```

## 查看 brew 的帮助
```
man brew
```
