---
title: Qt 应用程序的图标
date: 2016-11-16 14:22:39
tags: Qt
---
设置应用程序的图标，只需要修改 `.pro` 文件:

* Mac 使用 icns 图标: `ICON     = AppIcon.icns`
* Windows 使用 ico 图标: `RC_ICONS = AppIcon.ico`

> 图标和 .pro 文件在同一个目录即可
