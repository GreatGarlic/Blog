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