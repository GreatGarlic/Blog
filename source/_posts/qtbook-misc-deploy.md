---
title: Qt 程序简单打包
date: 2017-08-21 10:52:06
tags: QtBook
---

程序在开发工具例如 QtCreator 中运行没有问题，不少同学开发好后就直接把 xxx.exe 给用户使用，用户双击 xxx.exe 后提示错误，打开程序失败。很是奇怪: 程序在我的电脑里打开好好的，为什么到其他电脑上就不行了呢，是不是他的电脑有问题？不知道此同学有没有在自己电脑上双击过这个程序！

例如双击下面的 Gui.exe，提示找不到 libgcc_s_dw2-1.dll，那是因为 Qt 的程序运行的时候除了需要可执行程序本身外，还需要依赖一些其他的 dll，需要把这些 dll 一起打包给用户才行:

![](/img/qtbook/misc/deploy-win-1.png)

Qt 程序打包一般有 2 种方式，纯手动打包和半自动打包，下面以 Windows 中打包 Qt 程序 Gui.exe 为例进行介绍，环境如下:

* 安装 MinGW 的 Qt 5.9.1 到 **F** 盘
* DLL 目录: **F:\Qt\Qt5.9.1\5.9.1\mingw53_32\bin**
* Qt 的插件目录: **F:\Qt\Qt5.9.1\5.9.1\mingw53_32\plugins** <!--more-->

## 纯手动打包

1. 双击 Gui.exe

2. 弹出错误对话框，提示缺少 dll 如 libgcc_s_dw2-1.dll

3. 到上面说的 **DLL 目录** 中复制 libgcc_s_dw2-1.dll 到 Gui.exe 所在目录

4. 重复步骤 1 到 4 直到不在提示缺少 dll

5. 复制需要的插件到 Gui.exe 所在目录:

   没有插件时程序能运行起来了(在其他电脑还可能会提示缺少 paltforms 的 dll，但是安装了 Qt 的电脑里不会提示)，先不要开心，如果还使用了数据库，PNG 等，会发现访问不了数据库，PNG 显示有时候会有问题等，但是没有任何错误提示。这是因为程序很多时候还依赖 Qt 的插件，程序运行时缺少插件是不会提示的，这就需要我们凭经验去判断还需要哪些插件(看 pro 文件的 QT 配置了什么模块)

6. 把 Gui.exe 和上面的这些 dll 一起给用户，到此程序打包完成了

## 半自动打包

借助 Qt 提供的 **windeployqt.exe** 进行打包:

1. 命令行进入 Gui.exe 所在目录

2. 执行命令 `F:\Qt\Qt5.9.1\5.9.1\mingw53_32\bin\windeployqt.exe Gui.exe`

   > windeployqt.exe 后的参数是可执行文件的名字

3. 可以看到有很多 dll 和所有需要的 Qt 插件被复制到了 Gui.exe 所在目录

4. 双击 Gui.exe，提示缺少 dll 如 libgcc_s_dw2-1.dll

5. 到上面说的 **DLL 目录** 中复制 libgcc_s_dw2-1.dll 到 Gui.exe 所在目录

6. 重复步骤 4 到 6直到不在提示缺少 dll

7. 把 Gui.exe 和上面的这些 dll 一起给用户，到此程序打包完成了

> 半自动打包的方式比纯手动打包的方式方便了很多，能够自动为找到所有需要的 Qt 插件，但也有弊端，**会复制一些不一定需要的 dll 和文件**，例如 opengl32sw.dll，D3Dcompiler_43.dll，libGLESV2.dll，Qt5Svg.dll，translations，这些文件加起来有 +20M，如果对空间敏感，可以把他们删除，然后运行程序看看有没有问题(绝大多数时候没问题)，如果有问题，则把需要的再放回去即可。

Windows  下 Qt 程序的打包完成了，Mac 和 Linux 下打包请参考 Windows 的方式进行研究，更完善的打包方式请在 Qt 的帮助文档中搜索 **deployment**，查看相关帮助文档:

![](/img/qtbook/misc/deploy-win-2.png)

## 程序文件

附上程序打包后的文件列表作为参考:

```
├── Gui.exe
├── Qt5Core.dll
├── Qt5Gui.dll
├── Qt5Multimedia.dll
├── Qt5MultimediaWidgets.dll
├── Qt5Network.dll
├── Qt5OpenGL.dll
├── Qt5Widgets.dll
├── libEGL.dll
├── libgcc_s_dw2-1.dll
├── libstdc++-6.dll
├── libwinpthread-1.dll
├── audio
│   └── qtaudio_windows.dll
├── bearer
│   ├── qgenericbearer.dll
│   └── qnativewifibearer.dll
├── iconengines
│   └── qsvgicon.dll
├── imageformats
│   ├── qgif.dll
│   ├── qicns.dll
│   ├── qico.dll
│   ├── qjpeg.dll
│   ├── qsvg.dll
│   ├── qtga.dll
│   ├── qtiff.dll
│   ├── qwbmp.dll
│   └── qwebp.dll
├── mediaservice
│   ├── dsengine.dll
│   └── qtmedia_audioengine.dll
├── platforms
│   └── qwindows.dll
└── playlistformats
    └── qtmultimedia_m3u.dll
```

> 提示: audio, bearer, iconengines, imageformats, mediaservice, platforms, playlistformats 都是 Qt 的插件，他们和 Gui.exe 在同一级目录。如果感觉插件都放在 Gui.exe 的目录太乱，可以把它们放到 plugins 目录中，并在 main 函数中调用 `QApplication::addLibraryPath("plugins")` 即可。

