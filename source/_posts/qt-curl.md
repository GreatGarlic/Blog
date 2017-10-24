---
title: Qt 使用 curl
date: 2017-10-23 13:42:12
tags: Qt
---

Qt 已经提供了 QNetworkAccessManager 用于 Http 访问，[Qt 访问网络的 HttpClient](http://qtdebug.com/qt-httpclient/) 对其进行了简单封装，如下就可以进行 GET 请求:

```cpp
HttpClient("http://localhost:8080/device").get([](const QString &response) {
    qDebug() << response;
});
```

但是，在非 Qt 项目中就不能使用 QNetworkAccessManager 了，还有就是因为 curl 成熟、强大、跨平台，可能有些项目更希望使用 curl，所以在此以 **Windows MinGW 的 Qt 项目**为例，介绍 curl 的集成使用。<!--more-->

## 编译 curl

curl 没有提供编译好的库，需要自己编译，可按照下面的步骤进行(curl 相关的目录根据对应的版本好做出修改)

* Windows 中可使用 VS2013 编译 curl，所以当然是要先安装 VS2013

* 到 <https://curl.haxx.se/download.html> 下载 curl 源码，选择最新版吧，解压得到的压缩包保存到 `C:/curl-7.56.0`

* `Win + CMD` 打开命令行

* `"C:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\vcvars32.bat"` 设置 VS 的环境变量，只会当前 CMD 生效

* `cd C:\curl-7.56.0\winbuild`

* `nmake /f Makefile.vc mode=dll VC=12` 进行编译动态编译，也可以阅读 BUILD.WINDOWS.txt 查看更详细的信息

* 编译后的库文件保存到 `C:\curl-7.56.0\builds\libcurl-vc12-x86-release-dll-ipv6-sspi-winssl`，复制此目录到 C 盘根目录，重命名为 libcurl，目录结构如下:

  ```
  C:/
  └── libcurl
      ├── bin
      │   ├── curl.exe
      │   └── libcurl.dll
      ├── include
      │   └── curl
      │       ├── curl.h
      │       ├── curlver.h
      │       ├── easy.h
      │       ├── mprintf.h
      │       ├── multi.h
      │       ├── stdcheaders.h
      │       ├── system.h
      │       └── typecheck-gcc.h
      └── lib
          ├── libcurl.exp
          └── libcurl.lib
  ```

## 使用 curl

Qt Creator 中创建一个 Qt 项目，pro 文件里加上 curl 的库信息

```
INCLUDEPATH += C:/libcurl/include
LIBS += C:/libcurl/bin/libcurl.dll
```

> 这里不需要动态库的索引 .lib 文件，也不需要把 .lib 转换成 .a，直接使用 .dll 就可以了，MinGW 的编译器能够识别，和 VS 的编译器有些不一样。

接下来就可以使用 curl 发送 Http 请求了

```cpp
#include <QDebug>
#include <cstring>
#include <curl/curl.h>

// curl 读取到的数据保存到 std::string
size_t curlSaveResponseToStdString(void *contents, size_t size, size_t nmemb, std::string *s) {
    size_t newLength = size * nmemb;
    size_t oldLength = s->size();
    s->resize(oldLength + newLength);
    std::copy((char*)contents, (char*)contents+newLength, s->begin()+oldLength);

    return size * nmemb;
}

int main(int argc, char *argv[]) {
    // 初始化 curl
    CURL *curl = curl_easy_init();

    if (curl) {
        std::string response;
        curl_easy_setopt(curl, CURLOPT_URL, "http://www.qtdebug.com/html/data.json"); // 设置要访问的网址
        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, curlSaveResponseToStdString); // 告诉 curl 保存响应到 string 中
        curl_easy_setopt(curl, CURLOPT_WRITEDATA, &response); // 请求的响应保存到变量 response 中
        curl_easy_setopt(curl, CURLOPT_VERBOSE, 0L); // 0 不输出请求的详细信息，1 输出
        CURLcode code = curl_easy_perform(curl);

        if (code == CURLE_OK) {
            // std::cout << response << std::endl; // 中文乱码，因为 std::string 对中文的支持不好
            // qDebug() << QString::fromUtf8(response.data()); // response.data() 返回的是 UTF-8 的字节数据
            qDebug() << QString::fromStdString(response); // 使用 qDebug() 输出，UTF-8 的中文不会乱码
        }
    } else {
        qDebug() << "Error";
    }

    // 释放 curl 资源
    curl_easy_cleanup(curl);

    return 0;
}
```

编译正常，运行时程序异常结束，这是因为程序找不到 libcurl.dll，把 libcurl.dll 复制到编译出来的 exe 目录，再次运行程序，看到控制台输出

> "{\"name\": \"Alice\"}\n"

哈哈，curl 终于集成到我们的项目里了，至于 curl 的更多使用，请执行搜索相关教程即可，在这里就不赘述了。