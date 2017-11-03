---
title: Qt 全局快捷键
date: 2017-11-03 09:23:15
tags: Qt
---

> 全局快捷键: 按下快捷键后，不管程序是否当前正在使用的程序，它都能得到此快捷键的事件通知，例如 Windows 里按下 `Ctrl + Shift + A` 就可以使用 QQ 截图一样。

Qt SDK 没有自带设置全局快捷键的功能，需要自己实现，在 Github 上也有人开源了全局快捷键的库，例如 [QHotKey](https://github.com/xtuer/QHotkey)，可以直接在项目中使用。

QHotkey 有很多特点，能够满足绝大多数的需求:

* Works on Windows, Mac and X11
* Easy to use, can use `QKeySequence` for easy shortcut input
* Supports almost all common keys (Depends on OS & Keyboard-Layout)
* Allows direct input of Key/Modifier-Combinations
* Supports multiple QHotkey-instances for the same shortcut (with optimisations)
* Thread-Safe - Can be used on all threads (See section Thread safety)
* Allows usage of native keycodes and modifiers, if needed

下面就写个 HelloWorld 级别的程序，定义全局快捷键 `Alt+P` 唤出程序:

```cpp
#include <QHotkey>
#include <QApplication>

Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget) {
    ui->setupUi(this);

    QHotkey *hotkey = new QHotkey(QKeySequence("Alt+P"), true); // Alt 和 P 之间不能有空格
    qDebug() << "Is Registered: " << hotkey->isRegistered();

    connect(hotkey, &QHotkey::activated, [this](){
        this->show();
        this->raise();
        this->activateWindow();
        this->raise();
        QApplication::setActiveWindow(this);
        this->raise();
    });
}
```

<!--more-->

如果直接编译上面的程序当然是编译不通过了，还没有把 QHotkey 加入到项目中呢，按照下面几步就可以了:

1. **下载 QHotkey**: 访问 <https://github.com/xtuer/QHotkey> 进行下载
2. **解压 QHotkey**: 解压下载得到的文件，如果不喜欢，可以把其他文件删除，只有下面几个文件是必须的:
   ```
   QHotkey-lib
   ├── QHotkey
   │   ├── QHotkey
   │   ├── qhotkey.cpp
   │   ├── qhotkey.h
   │   ├── qhotkey_mac.cpp
   │   ├── qhotkey_p.h
   │   ├── qhotkey_win.cpp
   │   └── qhotkey_x11.cpp
   ├── qhotkey.prc
   └── qhotkey.pri
   ```
3. **引入 QHotkey**: 在项目的 pro 文件中使用 `include(${QHotKey-lib-Path}/qhotkey.pri)` 引入 QHotkey，编译运行程序，点击一下其他程序让我们的程序失去焦点，然后按下 `Alt+P`，如果我们的程序被激活为当前程序，说明全局快捷键功能生效了。

   > 有很直很直的同学估计会直接复制粘贴 `include(${QHotKey-lib-Path}/qhotkey.pri)` 到 pro 文件里，编译报错。说明一下吧，`${QHotKey-lib-Path}` 指的是 QHotkey 文件存放的路径，因为每个人保存的目录都不一样，所以用变量的样子来表示啦。

QHotkey 不但可以使用源文件的方式引入到项目中，还可以编译成动态链接库再使用：

1. Qt Creator 中打开 `${QHotKey-lib-Path}/QHotkey/QHotkey.pro` 
2. 编译，Mac 下编译后得到下面的这些库文件，并把头文件复制到一起(Windows 的参考这个就可以了，只是动态库的名字不一样而已)

   ```
   QHotkey
   ├── include
   │   ├── QHotkey
   │   └── qhotkey.h
   ├── libQHotkey.1.2.1.dylib
   ├── libQHotkey.1.2.dylib -> libQHotkey.1.2.1.dylib
   ├── libQHotkey.1.dylib -> libQHotkey.1.2.1.dylib
   └── libQHotkey.dylib -> libQHotkey.1.2.1.dylib
   ```
3. 在项目的 pro 文件里引入 QHotkey 的动态链接库(路径就不多说啥了，很直很直的同学自己去面壁)

   ```
   LIBS += -L/Users/Biao/Desktop/QHotkey -lQHotkey
   INCLUDEPATH += /Users/Biao/Desktop/QHotkey/include
   ```
4. 编译，并把上面得到的 QHotkey 的动态链接库文件复制到程序的可执行文件目录

   ```
   ├── libQHotkey.1.2.1.dylib
   ├── libQHotkey.1.2.dylib -> libQHotkey.1.2.1.dylib
   ├── libQHotkey.1.dylib -> libQHotkey.1.2.1.dylib
   └── libQHotkey.dylib -> libQHotkey.1.2.1.dylib
   ```
5. 运行，测试全局快捷键是否生效，如无操作失误，一切都在掌握之中

