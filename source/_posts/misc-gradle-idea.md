---
title: IDEA 创建 Gradle Module
date: 2016-06-29 10:09:48
tags: [Misc, Gradle]
---

以创建名为 SpringIntegration 的 Gradle Module 为例介绍在 IDEA 创建 Gradle Module 的步骤。

> IDEA 的 Maven Module 里修改 pom.xml 中的依赖后会自动下载相关的 Jar 包和源码，并添加到 classpath 里。
>
> 但是 IDEA 的 Gradle Module 里修改 build.gradle 中的依赖后却不会自动下载相关的 Jar 包和源码，并添加到 classpath 里，需要我们手动的在 Gradle 的工具窗口里点击刷新 Gradle Module 才会执行这些操作。

<!--more-->

## 1. 选择创建 Gradle Module
![](/img/misc/Gradle-1.png)

## 2. 输入 GroupId 和 ArtifactId
![](/img/misc/Gradle-2.png)

## 3. 配置 Gradle 的行为
* 创建标准的目录结构 (和 Maven 的一样)
* 使用本地安装的 Gradle

![](/img/misc/Gradle-3.png)

## 4. Module 存储的位置
![](/img/misc/Gradle-4.png)

## 5. 生成的项目结构
* 手动添加了 Spring 的依赖

![](/img/misc/Gradle-5.png)

## 6. 更新依赖
1. 点击红框里的按钮
2. 可以看到才会更新

![](/img/misc/Gradle-6.png)
