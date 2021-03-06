---
title: VS2013 使用 dll
date: 2017-10-24 10:48:20
tags: Qt
---

[Qt 使用 curl](http://qtdebug.com/qt-curl) 一文中介绍了怎么编译 curl 并且在 Qt 项目中使用，那么在 VS 项目中应该怎么使用 curl 的 dll 呢？

动态库的使用分为隐式链接和显示链接两种方式:

* **显示链接**: 只需要 `.dll` 动态库文件，代码中使用 `LoadLibrary + GetProcAddress` 加载函数后需要自己进行函数类型转换：

  ```cpp
  // 函数类型定义
  typedef void (*DLLFunc)(int); 

  // 加载 dll 中的函数
  HINSTANCE hInstLibrary = LoadLibrary("DLLSample.dll");
  DLLFunc dllFunc = (DLLFunc)GetProcAddress(hInstLibrary, "TestDLL");

  // 执行函数
  dllFunc(123);
  ```

* **隐式链接**: 需要 `.h` 头文件、`.lib` 库导入文件和 `.dll` 动态库文件，代码中直接使用库的函数即可

  > 推荐使用隐式链接，更省事，可参考 [LIB 和 DLL 的区别与使用](http://www.cppblog.com/biao/archive/2013/03/14/198416.html)。

VS2013 中隐式链接使用 dll 一般有两种方法:

* 使用 `#pragma` 引入 lib

* 设置 `项目属性` 引入 lib<!--more-->


下面就以隐式链接使用 curl 的 dll 为例进行介绍，curl 存放在 `C:/libcurl`。

## 创建 Visual C++ 控制台程序

![](/img/qt/vs-dll-1.png)

```cpp
#include "stdafx.h"

int _tmain(int argc, _TCHAR* argv[]) {
    return 0;
}
```

## 一：使用 `#pragma` 引入 lib

包含头文件以及使用 `#pragma` 引入 lib 就可以了：

```cpp
#include "stdafx.h"

// 包含 curl 的头文件和 lib(可以把库放到项目中，使用相对路径引入)
#include "C:/libcurl/include/curl/curl.h"
#pragma comment(lib, "C:/libcurl/lib/libcurl.lib")

int _tmain(int argc, _TCHAR* argv[]) {
    CURL *curl = curl_easy_init();

    // curl 访问网络的代码请参考 Qt 使用 curl 的例子
    if (curl) {
        ...
    }

    curl_easy_cleanup(curl);

    return 0;
}
```

把 libcurl.dll 复制到项目编译出的 exe 目录，运行项目，可以看到 curl 访问网络成功。

## 二：设置 `项目属性` 引入 lib

在项目的名字上 `右键 > Properties`，在弹出的对话框中配置需要的 `头文件目录`、`lib 目录` 和  `lib 名字`：

* 头文件目录: `C/C++ > General > Additional Include Directories`

  ![](/img/qt/vs-dll-2.png)

* lib 目录: `Linker > General > Additinal Library Directories`

  ![](/img/qt/vs-dll-3.png)

* lib 的名字: `Linker > Input > Additional Dependencies`

  ![](/img/qt/vs-dll-4.png)

通过上面三步就配置好了动态库，然后再程序中包含头文件就可以使用了:

```cpp
#include "stdafx.h"

// 包含 curl 的头文件和 lib
#include <curl/curl.h>

int _tmain(int argc, _TCHAR* argv[]) {
    CURL *curl = curl_easy_init();

    // curl 访问网络的代码请参考 Qt 使用 curl 的例子
    if (curl) {
        ...
    }

    curl_easy_cleanup(curl);

    return 0;
}
```

把 libcurl.dll 复制到项目编译出的 exe 目录，运行项目，可以看到 curl 访问网络成功。

> 通过比较，是不是感觉使用 `#pragma` 的方式更简单一些？