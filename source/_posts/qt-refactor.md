---
title: QtCreator 中重构 Widget 的名字
date: 2017-07-16 11:04:34
tags: Qt
---

QtCreator 中创建的 Qt Designer Form Class 包含三个文件: **.h**, **.cpp**, **.ui**，例如我们创建了一个 Form Class **Widget**，则包含下面三个文件：Widget.h, Widget.cpp, Widget.ui，其中的类名为 Widget，如果想要把其重命名为 **MyWidget**，则可以按照下面几步进行：

* 文件重命名为 MyWidget.h, MyWidget.cpp, MyWidget.ui
* 修改 MyWidget.ui 中的 objectName
* 重构 MyWidget.h 中的类名 Ui::Widget 和 Widget，同时也可修改 #ifndef 的名字
* 修改 MyWidget.cpp 中的 #include

<!--more-->

## 文件重命名

QtCreator 中文件名上 **右键** > **rename**，重命名文件名为:

* MyWidget.h
* MyWidget.cpp
* MyWidget.ui

![](/img/qt/rename-file.png)

> QtCreator 中的 rename 修改文件名的同时会:
>
> * 修改 pro 文件里的 HEADERS，SOURCES，FORMS
> * 把其他文件中 #include "Widget.h" 自动修改为 #include "MyWidget.h"
> * 但是 #include "ui_Widget.h" 不会自动修改为 #include "ui_MyWidget.h"，这个需要我们自己手动修改

## MyWidget.ui

MyWidget.ui 中修改 objectName 为 MyWidget

![](/img/qt/rename-ui-designer.png)

## MyWidget.h

* WIDGET_H 修改为 MYWIDGET_H

* namespace Ui 中 Widget 重构为 MyWidget

  ![](/img/qt/rename-class-ui.png)

* class Widget 重构为 class MyWidget

  ![](/img/qt/rename-class.png)

## MyWidget.cpp

修改 `#include "ui_Widget.h"` 为 `#include "ui_MyWidget.h"`