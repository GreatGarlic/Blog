---
title: Qt5 乱码与 BOM
date: 2016-12-28 10:46:31
tags: Qt
---

虽然 Qt5 强制要求源码必须使用 UTF-8 的编码，但是仍然会遇到乱码问题，例如在 MinGW 下编译没问题的工程使用 VS 的编译器编译时有可能报错，MinGW 编译的程序运行时中文正常显示，但是 VS 环境下却是乱码等，下面来解决 Qt5 的乱码问题。<!--more-->

## BOM 问题

![](/img/qt/BOM.png)

使用 VS 的编译器编译 Qt 项目时有可能会遇到像上面这样的警告和错误，一般都是因为程序的源文件 .h 和 .cpp 文件的编码虽然是 UTF-8 的，但是 `缺少 BOM`，这个问题也很好解决，给程序的源文件都加上 BOM 就可以了。给文件添加 BOM 的方法也很多，例如使用 Notepad++ 一个一个的打开所有文件，然后都保存为带 BOM 的 UTF-8 就可以了，不过这样做的效率显然不高，这里提供一个可以递归的给指定目录下所有指定后缀名的文件批量添加和删除 BOM 的工具，也是一个 Qt 的工程，请自行下载编译: [Bomer 的源码](/download/Bomer.7z)

## 中文乱码

下面的程序只有一个按钮，按钮的文本为 “你好”，使用 MinGW 编译运行，程序没有乱码问题。

```cpp
#include <QApplication>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    QPushButton b("你好");
    b.show();

    return a.exec();
}
```

然后使用 VS 编译器编译运行，编译没问题的情况下(如有 BOM 问题，请按上面方法先解决)，运行后按钮的文字出现了乱码，有 2 个解决方法:

* 中文字符串使用 QStringLiteral 创建

  ```cpp
  #include <QApplication>
  #include <QPushButton>

  int main(int argc, char *argv[]) {
      QApplication a(argc, argv);

      QPushButton b(QStringLiteral("你好")); // 瞅这里
      b.show();

      return a.exec();
  }
  ```

  再次编译运行，乱码问题解决了。这种方法在 MinGW 和 VS 中都没问题，但是仔细考虑一下，程序中如果有成百上千的中文字符串，每个地方都要修改，工作量大到让人崩溃。

* 使用 VS 的时候在 main() 函数前添加 pragma 指定程序运行的编码即可: 

  ```cpp
  #include <QApplication>
  #include <QPushButton>

  #pragma execution_character_set("utf-8") // 瞅这里

  int main(int argc, char *argv[]) {
      QApplication a(argc, argv);

      QPushButton b("你好");
      b.show();

      return a.exec();
  }
  ```

  这个方法不错，不需要把中文字符串都使用 QStringLiteral 创建就解决了 VS 中的乱码问题，但是在非 VS 的编译器环境下不识别 execution_character_set 这个 pragma，编译时会给出一个警告，虽然不影响编译运行，但是看上去总是有点不舒服，删除即可，一行代码而已。