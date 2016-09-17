---
title: Qt 访问网络
date: 2016-09-13 11:45:54
tags: Qt
---

Qt 中访问网络使用 `QNetworkAccessManager`，它的 API 是异步，这样在访问网络的时候不需要启动一个线程，在线程里执行请求的代码。

需要注意一点的是，请求响应的对象 `QNetworkReply` 需要我们自己手动的删除，一般都会在 `QNetworkAccessManager::finished` 信号的曹函数里使用 `reply->deleteLater()` 删除，不要直接 `delete reply`。

本文的最终结果为实现调用一个函数就能访问网络:

```cpp
QNetworkAccessManager *manager = new QNetworkAccessManager();

NetworkUtil::get(manager, "http://www.baidu.com", [](const QString &response) {
    qDebug() << response;
});
```

<!--more-->

## 例一

```cpp
#include <QDebug>
#include <QApplication>
#include <QNetworkRequest>
#include <QNetworkReply>
#include <QNetworkAccessManager>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QNetworkAccessManager *manager = new QNetworkAccessManager();
    QNetworkRequest request(QUrl("http://www.baidu.com"));
    QNetworkReply *reply = manager->get(request);

    int count = 0;

    QObject::connect(reply, &QNetworkReply::readyRead, [&] {
        qDebug() << QString(reply->readAll());
        qDebug() << ++count;
    });
    
    // 请求错误处理
    QObject::connect(reply, static_cast<void (QNetworkReply::*)(QNetworkReply::NetworkError)>(&QNetworkReply::error), [&] {
        qDebug() << reply->errorString();
    });

    // 请求结束时删除 reply 释放内存
    QObject::connect(reply, &QNetworkReply::finished, [&] {
        reply->deleteLater();
    });

    return app.exec();
}
```

## 例二
仔细观察上面程序的输出结果，由于返回的数据比较大，`readyRead 被调用了多次`，而不是一次性就得到了请求的响应数据，这个特点在某些情况下很有用，例如下载 100M 的文件，多次读取肯定是合适的，因为读取后数据就会从 reply 中删除，不会导致占用太多内存，但是在某些情况下却不太好用，例如读取一个响应 JSON 的数据，一般都不会太大，大的也就几十上百 K，如果一次得不到 JSON 的全部数据，多次读取的情况下想要拼出一个完整的 JSON 字符串不太容易，这时如果能一次性的得到响应的 JSON 数据是不是就很方便了呢？

要一次性读取到响应的数据可以在 `QNetworkReply::finished` 信号处理中进行，如下

```cpp
int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QNetworkAccessManager *manager = new QNetworkAccessManager();
    QNetworkRequest request(QUrl("http://www.baidu.com"));
    QNetworkReply *reply = manager->get(request);

    // 请求错误处理
    QObject::connect(reply, static_cast<void (QNetworkReply::*)(QNetworkReply::NetworkError)>(&QNetworkReply::error), [&] {
        qDebug() << reply->errorString();
    });
    
    // 请求结束时一次性读取所有响应数据
    QObject::connect(reply, &QNetworkReply::finished, [&] {
        if (reply->error() == QNetworkReply::NoError) {
            qDebug() << reply->readAll();
        }

        reply->deleteLater();
    });

    return app.exec();
}
```

## 优化
观察上面的程序，会发现很多代码都是重复的模版代码，例如

* 创建 QNetworkRequest
* 获取 QNetworkReply
* 删除 QNetworkReply
* 错误处理
* 响应处理

大量的模版代码可以把它们封装成一个工具类，方便使用，参考下面 `main()` 函数里的调用，代码一下子看上去就清晰简单了很多。

`main.cpp`

```cpp
#include "NetworkUtil.h"

#include <QDebug>
#include <QApplication>
#include <QNetworkAccessManager>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QNetworkAccessManager *manager = new QNetworkAccessManager();

    // 访问 baidu
    NetworkUtil::get(manager, "http://www.baidu.com", [](const QString &response) {
        qDebug() << response;
    });

    // 访问 163
    NetworkUtil::get(manager, "http://www.163.com", [](const QString &response) {
        qDebug() << response;
    }, NULL, "GB2312");

    return app.exec();
}
```

`NetworkUtil.h`

```cpp
#ifndef NETWORKUTIL_H
#define NETWORKUTIL_H

#include <functional>

class QString;
class QNetworkAccessManager;

class NetworkUtil {
public:
    /**
     * @brief 使用 GET 访问网络
     *
     * @param manager QNetworkAccessManager 对象
     * @param url 需要访问的 URL
     * @param successHandler 访问成功的 Lambda 回调函数
     * @param errorHandler 访问失败的 Lambda 回调函数
     * @param encoding 读取响应数据的编码
     */
    static void get(QNetworkAccessManager *manager,
                    const QString &url,
                    std::function<void (const QString &)> successHandler,
                    std::function<void (const QString &)> errorHandler = NULL,
                    const char *encoding = "UTF-8");
};

#endif // NETWORKUTIL_H
```

`NetworkUtil.cpp`

```cpp
#include "NetworkUtil.h"
#include <QNetworkRequest>
#include <QNetworkReply>
#include <QNetworkAccessManager>
#include <QTextStream>

void NetworkUtil::get(QNetworkAccessManager *manager,
                      const QString &url,
                      std::function<void (const QString &)> successHandler,
                      std::function<void (const QString &)> errorHandler,
                      const char *encoding) {
    QUrl urlx(url);
    QNetworkRequest request(urlx);
    QNetworkReply *reply = manager->get(request);

    // 请求错误处理
    QObject::connect(reply, static_cast<void (QNetworkReply::*)(QNetworkReply::NetworkError)>(&QNetworkReply::error), [=] {
        if (NULL != errorHandler) {
            errorHandler(reply->errorString());
        }
    });

    // 请求结束时一次性读取所有响应数据
    QObject::connect(reply, &QNetworkReply::finished, [=] {
        if (reply->error() == QNetworkReply::NoError) {
            // 读取响应数据
            QTextStream in(reply);
            QString result;

            in.setCodec(encoding);
            while (!in.atEnd()) {
                result += in.readLine();
            }

            successHandler(result);
        }

        reply->deleteLater();
    });
}
```

## 思考
工具类 `NetworkUtil` 介绍了一次性读取时 Get 的封装，大家思考一下 Get 多次读取的封装，Post 请求的封装等。

## 挑战
线程 MyThread 每隔 2 秒发出信号通知 MyWidget 访问 <http://www.baidu.com>，然后把响应的数据显示到 MyWidget 上的 QTextEdit 中。

不是说 `QNetworkAccessManager` 的 API 是异步的么，为啥这里又需要用线程了？曾经遇到这么一个需求，连接身份证读卡器，不停自动的读取身份证信息，上传到 Web 服务器，也就是这个挑战能使用到的场景，你可以把它换成不停的读取 NFC 卡，刷门禁卡等。因为是不停的读卡，所以需要在线程中读卡，否则界面会被冻结掉。


