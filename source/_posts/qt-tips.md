---
title: Qt Tips
date: 2016-12-14 17:37:18
tags: Qt
---

## 无边框对话框不在任务栏显示图标

```cpp
QDialog *dlg = new QDialog(NULL, Qt::Dialog | Qt::Popup | Qt::FramelessWindowHint);
dlg->show();
```

> Qt::Popup 是个小技巧，只有先点击对话框后才能点击对话框后面的 widget

<!--more-->

## Xcode 升级后报错

Xcode 升级 MacOSX10.12.sdk 为 MacOSX10.13.sdk 后，QtCreator 中编译项目报错如下:

```
Warning: no such sysroot directory: '/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk'
Error: 'TargetConditionals.h' file not found
```

解决这个问题，需要修改 `<QtInstallDir>/clang_64/mkspecs/qdevice.pri` 中的 QMAKE_MAC_SDK 为:

```
# 修改前为 QMAKE_MAC_SDK = macosx10.12
QMAKE_MAC_SDK = macosx10.13
```

> 如果重新安装 Qt 也是没有问题的。

