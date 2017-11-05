---
title: 创建使用动态链接库
date: 2017-11-05 08:48:41
tags: QtBook
---

想一想大多数时候我们的项目是不是所有代码都会放在同一个工程中？人少的时候都不是事，但当项目越来越大，开发人员越来越多，会发觉开发、管理能让人窒息，大家都绞在一起，出问题时互相推诿责任，各自有理，这时如果按照功能模块进行分组各自开发，以库的形式提供给其他人使用，就能够最大限度的并行开发，提高工作效率，而且项目的模块也很清晰，责任一目了然，此外使用动态链接库后还能够按模块升级，编译的速度也更快。下面就介绍怎么在工程中创建和使用动态链接库。

> Windows 中叫动态链接库(Lynamic Link Library: .dll)，Linux 中叫共享库(Shared Library: .so)，Mac 下后缀为 .dylib，实际指的是一种类型的库，这里都统称为动态链接库吧。

动态链接库需要理解符号的概念，符号包含函数、变量或者类，分为公有符号和私有符号:

* 公有符号: 在其他程序或者库使用的符号，需要根据用途进行特殊标记:

  * `Q_DECL_EXPORT`: 编译为动态链接库时符号要标记为 `Q_DECL_EXPORT`，表明是导出符号
  * `Q_DECL_IMPORT`: 在调用动态链接库时符号要标记为 `Q_DECL_IMPORT`，表明是使用符号

* 私有符号: 除了公有符号外的其他符号，在此库外不应该能够访问，不需要进行标记

  > 建议: 不要在头文件中声明私有符号。

符号上的标记 `Q_DECL_EXPORT` 和 `Q_DECL_IMPORT` 不能同时存在，为了在导出和导入时使用同一个头文件， 头文件中包含下面的宏，编译时根据条件使用不同的宏就可以了:

```cpp
#include <QtCore/qglobal.h>

// 根据条件定义 MYLIBRARY_SHARED_SYMBOL 为不同的宏
#if defined(MYLIBRARY_LIBRARY)
#   define MYLIBRARY_SHARED_SYMBOL Q_DECL_EXPORT
#else
#   define MYLIBRARY_SHARED_SYMBOL Q_DECL_IMPORT
#endif

// 使用 MYLIBRARY_SHARED_SYMBOL 声明符号，编译时会根据条件替换为 Q_DECL_EXPORT 或者 Q_DECL_IMPORT
class MYLIBRARY_SHARED_SYMBOL Calculator {
   ...
};
```

动态链接库工程的 pro 文件中添加 `DEFINES += MYLIBRARY_LIBRARY`，编译的时候 `MYLIBRARY_SHARED_SYMBOL` 被替换为 `Q_DECL_EXPORT`，使用此动态链接库的工程的 pro 文件中千万不要加 `DEFINES += MYLIBRARY_LIBRARY`，编译的时候 `MYLIBRARY_SHARED_SYMBOL` 就被替换为 `Q_DECL_IMPORT`，达到了在导出和导入时使用同一个头文件的目的。<!--more-->

说了这么多，还是不知道 Qt 中怎么创建和使用动态链接库，下面就以使用子工程的方式介绍创建和使用动态链接库，工程结构如下:

```
SharedLibraryTest
   ├── MyLibrary
   │   ├── Calculator.cpp
   │   ├── Calculator.h
   │   └── MyLibrary.pro
   ├── MyProject
   │   ├── MyProject.pro
   │   └── main.cpp
   └── SharedLibrary.pro
```

SharedLibraryTest 是 Subdirs Project，包含动态链接库的工程(MyLibrary)和使用动态链接库的工程(MyProject)。

## 创建工程

Qt Creator 中创建工程的步骤如下:

1. 创建工程 SharedLibraryTest:
   1. `File > New File or Project... > Other Project > Subdirs Project`
   2. 点击 `Choose...`
   3. 输入工程名 `SharedLibraryTest` 然后创建
2. 创建工程 MyLibrary:
   1. `工程 SharedLibraryTest 上右键 > New Subproject... > Library > C++ Library`
   2. 点击 `Choose...`
   3. 选择 Type 为 `Shared Library`
   4. 输入工程名 `MyLibrary` 然后创建
3. 创建工程 MyProject:
   1. `工程 SharedLibraryTest 上右键 > New Subproject... > Application > Qt Console Application`
   2. 点击 `Choose...`
   3. 输入工程名 `MyProject` 然后创建

## 修改 MyLibrary

为了清晰起见，我们先把 MyLibrary 下的 .h 和 .cpp 文件都删除掉，创建 C++ Class Calculator 得到 Calculator.h 和 Calculator.cpp，然后编辑它们:

```cpp
// 文件名: Calculator.h
#ifndef CALCULATOR_H
#define CALCULATOR_H

#include <QtCore/qglobal.h>

#if defined(MYLIBRARY_LIBRARY)
#   define MYLIBRARY_SHARED_SYMBOL Q_DECL_EXPORT
#else
#   define MYLIBRARY_SHARED_SYMBOL Q_DECL_IMPORT
#endif

class MYLIBRARY_SHARED_SYMBOL Calculator {
public:
    Calculator();
    int add(int a, int b) const;
};

void MYLIBRARY_SHARED_SYMBOL work();

#endif // CALCULATOR_H
```

```cpp
// 文件名: Calculator.cpp
#include "Calculator.h"
#include <QDebug>

Calculator::Calculator() {

}

int Calculator::add(int a, int b) const {
    return a + b;
}

void doWork() {
    qDebug() << "work() -> doWork()";
}

void work() {
    doWork();
}
```

> 提示: `class Calculator` 和 `work()` 是公有符号，在头文件里声明，可以在其他程序中使用，而 `doWork()` 是私有符号，不在头文件里声明，对于其他程序是不可访问的。

因为这个工程是生成动态链接库的，所以需要在工程的 pro 文件中增加 `DEFINES += MYLIBRARY_LIBRARY` 这一句，这样编译时 `MYLIBRARY_SHARED_SYMBOL` 为 `Q_DECL_EXPORT`，工程的 pro 文件如下:

```ini
#-------------------------------------------------
#
# Project created by QtCreator 2017-11-05T08:33:53
#
#-------------------------------------------------

QT      -= gui

TARGET   = MyLibrary
TEMPLATE = lib

DEFINES += MYLIBRARY_LIBRARY

# Output directory
CONFIG(debug, debug|release) {
    output = debug
}
CONFIG(release, debug|release) {
    output = release
}

DESTDIR     = ../bin
OBJECTS_DIR = $$output
MOC_DIR     = $$output
RCC_DIR     = $$output
UI_DIR      = $$output

SOURCES += \
    Calculator.cpp

HEADERS += \
    Calculator.h

unix {
    target.path = /usr/lib
    INSTALLS += target
}
```

下面的部分不是必须的，只是我常用来组织编译输出的目录:

```ini
CONFIG(debug, debug|release) {
    output = debug
}
CONFIG(release, debug|release) {
    output = release
}

DESTDIR     = ../bin
OBJECTS_DIR = $$output
MOC_DIR     = $$output
RCC_DIR     = $$output
UI_DIR      = $$output
```

## 修改 MyProject

```cpp
// 文件名: main.cpp
#include <Calculator.h>
#include <QDebug>

int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    // 调用库中的函数
    work();

    // 生成库中的类对象
    Calculator calculator;
    qDebug() << calculator.add(2, 3);

    return 0;
}
```

工程的 pro 文件如下，没有 `DEFINES += MYLIBRARY_LIBRARY` 这一句哦，因为这个工程是使用动态链接库的，`MYLIBRARY_SHARED_SYMBOL` 应该为  `Q_DECL_IMPORT`:

```ini
QT -= gui

CONFIG += c++11 console
CONFIG -= app_bundle

INCLUDEPATH += $$PWD/../MyLibrary
LIBS += -L$$OUT_PWD/../bin -lMyLibrary

# Output directory
CONFIG(debug, debug|release) {
    output = debug
}
CONFIG(release, debug|release) {
    output = release
}

DESTDIR     = ../bin
OBJECTS_DIR = $$output
MOC_DIR     = $$output
RCC_DIR     = $$output
UI_DIR      = $$output

SOURCES += main.cpp
```

`INCLUDEPATH += $$PWD/../MyLibrary` 添加 `MyLibrary` 的路径到包含目录中，使用的时候就可以  `include <Calculator.h>` 这样包含头文件了， `LIBS += -L$$OUT_PWD/../bin - lMyLibrary` 则是引入工程 MyLibrary 生成的动态链接库。

编译、运行工程，控制台输出:

> work() -> doWork()
> 5

可以看到编译输出目录生成了动态链接库，并且在工程中也成功使用了，以后复杂的项目就可以按模块进行开发了。当然也可以不必使用 Subdirs Project 的方式，而是每个模块一个工程，pro 文件进行简单的修改即可。

## 参考资料

* Qt 的帮助文件中搜索: `Creating Shared Libraries` ，查看怎么创建动态链接库
* Qt 的帮助文件中搜索: `Third Party Libraries`，查看怎么使用第三方的动态链接库

