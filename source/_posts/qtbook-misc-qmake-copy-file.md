---
title: qmake 时复制文件
date: 2016-11-16 12:50:55
tags: QtBook
---
有时在编译前需要准备一些文件，例如修改了 QtCreator 的编译输出目录: `Build & Run > Default build directory`，使用 Promote 后需要在编译前把相应 Widget 的头文件复制到 `.o` 文件所在的目录，这时就可以在 `.pro` 文件中使用复制文件的命令(其实就是执行系统命令)，让 qmake 执行这些命令来复制文件，而不是手动的复制需要的文件。

可以使用 qmake 时能执行系统的命令的特性来做很多事情，不只是复制文件。

<!--more-->

```js
QT += core gui sql xml

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

TARGET   = Schedule
TEMPLATE = app
CONFIG  -=app_bundle

# Output directory
CONFIG(debug, debug|release) {
    compiled = debug
}
CONFIG(release, debug|release) {
    compiled = release
}

# All temporay files are all put in the directory $$compiled
DESTDIR     = bin
OBJECTS_DIR = $$compiled
MOC_DIR     = $$compiled
RCC_DIR     = $$compiled
UI_DIR      = $$compiled

# Copy promotion required headers to build directory
win32 {
    COPY_DEST = $$replace(OUT_PWD, /, \\)
    system("copy gui\\ClassWidget.h   $$COPY_DEST\\$$compiled\\ClassWidget.h")
    system("copy gui\\CourseWidget.h  $$COPY_DEST\\$$compiled\\CourseWidget.h")
    system("copy gui\\TeacherWidget.h $$COPY_DEST\\$$compiled\\TeacherWidget.h")
}

mac {
    system("cp gui/ClassWidget.h   $$OUT_PWD/$$compiled/ClassWidget.h")
    system("cp gui/CourseWidget.h  $$OUT_PWD/$$compiled/CourseWidget.h")
    system("cp gui/TeacherWidget.h $$OUT_PWD/$$compiled/TeacherWidget.h")
}

SOURCES += main.cpp
```

> 在 .pro 文件里路径分隔符都是 /，但在 Windows 中, / 对于 copy 命令有特殊用途，所以需要把 / 替换为 \，否则复制文件会失败
