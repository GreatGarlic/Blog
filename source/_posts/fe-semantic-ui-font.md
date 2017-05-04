---
title: 修改 Semantic UI 的默认字体
date: 2017-05-03 17:21:01
tags: [FE, Semantic-Ui]
---

Semantic Ui 默认使用的是谷歌提供的字体，并且是直接使用了谷歌的官方链接。谷歌网站在国内访问速度很差，甚至根本无法访问，需要对 Semantic UI 的源文件进行一下手动修改:

1. 使用 Nodejs 下载 Semantic Ui 源码: http://www.semantic-ui.cn/introduction/getting-started.html

   * 安装 Nodejs
   * 安装 gulp: **npm install -g gulp**
   * 下载 Semantic Ui: **npm install semantic-ui --save**

2. 修改 **src\themes\default\globals\site.variables**

   * 修改文件中的 **@fontName** 的值来设置 Semantic UI 的默认字体，这里使用了**微软雅黑: Microsoft YaHei**
   * 修改文件中的 **@importGoogleFonts 为 false** 禁止使用 Google 的字体

3. 使用命令 **gulp build** 编译一下 Semantic UI

4. 复制生成的 dist 目录中的文件到项目里即可<!--more-->

![](/img/fe/semantic-ui-font.png)

参考: <http://www.cnblogs.com/xwgli/p/5551139.html>