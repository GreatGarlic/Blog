---
title: Qt 项目中使用 OpenCV
date: 2018-02-05 15:57:24
tags: Qt
---

相信很多人在 Qt 项目中使用 OpenCV 都遇到过麻烦，[Windows 开发者软件推荐](http://qtdebug.com/windows-develop-software/) 一文中介绍过使用 Vcpkg 来管理第三方库，这里就使用 Vcpkg 安装 OpenCV 然后在 `MSVC 的 Qt` 项目中使用(因为 Vcpkg 使用的是 MSVC 编译器，OpenCV 是 C++ 库，不能够跨编译器，所以 MinGW 的项目不能使用):

1. 安装 Vcpkg 就不用多说了，安装到 C 盘根目录下吧

2. 安装 OpenCV: `vcpkg install opencv`

3. Qt Creator 中 创建 Qt 项目

4. 修改项目的 .pro 文件，主要是下面 2 句引入 OpenCV

   ```
   INCLUDEPATH += C:/vcpkg/installed/x86-windows/include
   LIBS        += C:/vcpkg/installed/x86-windows/lib/opencv_*.lib
   ```

   > 使用 `opencv_*.lib` 引入所有 `opencv_` 开头的 lib 文件，这样就不需要一个一个的引入 lib 了。
   >
   > 引入 dll 的时候也可以使用 `*` 来匹配一次引入多个，例如 `tiff*.dll`。

5. main.cpp

   ```cpp
   #include <opencv2/opencv.hpp>
   using namespace cv;

   int main()  {
       Mat img = imread("D:/Wallpaper/desktop.jpg");
       imshow("TEST", img);
       waitKey(6000);

       return 0;
   }
   ```

6. 把 OpenCV 相关的 DLL 从 `C:/vcpkg/installed/x86-windows/bin` 目录复制到编译出的 exe 所在目录

7. 运行程序，然后就看到打开一个窗口，图片显示在窗口中


