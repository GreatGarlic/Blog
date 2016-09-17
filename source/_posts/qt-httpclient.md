---
title: Qt 访问网络的 HttpClient
date: 2016-09-17 19:51:56
tags: Qt
---

Qt 使用 `QNetworkAccessManager` 访问网络，这里对其进行了简单的封装，访问网络的代码可以简化为:

```cpp
HttpClient("http://localhost:8080/device").get([](const QString &response) {
    qDebug() << response;
});
```

<!--more-->

HttpClient 的实现为下面列出的 `HttpClient.h` 和 `HttpClient.cpp`

## main.cpp
`main()` 函数里展示了 HttpClient 的使用示例。

```cpp
#include "HttpClient.h"

#include <QDebug>
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    {
        // 在代码块里执行网络访问，是为了测试 HttpClient 对象在被析构后，网络访问的回调函数仍然能正常执行
        // GET 请求无参数
        HttpClient("http://localhost:8080/device").get([](const QString &response) {
            qDebug() << response;
        });

        // GET 请求有参数，有自定义 header
        HttpClient("http://localhost:8080/signIn")
                .addParam("id", "1")
                .addParam("name", "诸葛亮")
                .addHeader("token", "123AS#D")
                .get([](const QString &response) {
            qDebug() << response;
        });

        // POST 请求有参数，有自定义 header
        HttpClient("http://localhost:8080/signIn")
                .addParam("id", "2")
                .addParam("name", "卧龙")
                .addHeader("token", "DER#2J7")
                .addHeader("content-type", "application/x-www-form-urlencoded")
                .post([](const QString &response) {
            qDebug() << response;
        });
    }

    qDebug() << "------";
    return app.exec();
}
```

## HttpClient.h
```cpp
#ifndef HTTPCLIENT_H
#define HTTPCLIENT_H

#include <functional>
#include <QHash>
#include <QUrlQuery>

class HttpClient {
public:
    HttpClient(const QString &url);
    ~HttpClient();

    /**
     * @brief 增加参数
     * @param name  参数的名字
     * @param value 参数的值
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& addParam(const QString &name, const QString &value);

    /**
     * @brief 增加访问头
     * @param header 访问头的名字
     * @param value  访问头的值
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& addHeader(const QString &header, const QString &value);

    /**
     * @brief 执行 GET 请求
     * @param successHandler 请求成功的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     * @param encoding       请求响应的编码
     */
    void get(std::function<void (const QString &)> successHandler,
             std::function<void (const QString &)> errorHandler = NULL,
             const char *encoding = "UTF-8");

    /**
     * @brief 执行 POST 请求
     * @param successHandler 请求成功的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     * @param encoding       请求响应的编码
     */
    void post(std::function<void (const QString &)> successHandler,
             std::function<void (const QString &)> errorHandler = NULL,
             const char *encoding = "UTF-8");

private:
    /**
     * @brief 执行请求的辅助函数
     * @param posted 为 true 表示 POST 请求，为 false 表示 GET 请求
     * @param successHandler 请求成功的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     * @param encoding       请求响应的编码
     */
    void execute(bool posted,
                 std::function<void (const QString &)> successHandler,
                 std::function<void (const QString &)> errorHandler,
                 const char *encoding);

    QString url; // 请求的 URL
    QUrlQuery params; // 请求的参数
    QHash<QString, QString> headers; // 请求的头
};

#endif // HTTPCLIENT_H
```

## HttpClient.cpp
```cpp
#include "HttpClient.h"

#include <QNetworkReply>
#include <QNetworkRequest>
#include <QNetworkAccessManager>
#include <QDebug>

HttpClient::HttpClient(const QString &url) : url(url) {
    qDebug() << "HttpClient";
}

HttpClient::~HttpClient() {
    qDebug() << "~HttpClient";
}

// 增加参数
HttpClient &HttpClient::addParam(const QString &name, const QString &value) {
    params.addQueryItem(name, value);
    return *this;
}

// 增加访问头
HttpClient &HttpClient::addHeader(const QString &header, const QString &value) {
    headers[header] = value;
    return *this;
}

// 执行 GET 请求
void HttpClient::get(std::function<void (const QString &)> successHandler,
                     std::function<void (const QString &)> errorHandler,
                     const char *encoding) {
    execute(false, successHandler, errorHandler, encoding);
}

// 执行 POST 请求
void HttpClient::post(std::function<void (const QString &)> successHandler,
                      std::function<void (const QString &)> errorHandler,
                      const char *encoding) {
    execute(true, successHandler, errorHandler, encoding);
}

// 执行请求的辅助函数
void HttpClient::execute(bool posted,
                         std::function<void (const QString &)> successHandler,
                         std::function<void (const QString &)> errorHandler,
                         const char *encoding) {
    // 如果是 GET 请求，并且参数不为空，则编码请求的参数，放到 URL 后面
    if (!posted && !params.isEmpty()) {
        url += "?" + params.toString(QUrl::FullyEncoded);
    }

    QUrl urlx(url);
    QNetworkRequest request(urlx);

    // 把请求的头添加到 request 中
    QHashIterator<QString, QString> iter(headers);
    while (iter.hasNext()) {
        iter.next();
        request.setRawHeader(iter.key().toUtf8(), iter.value().toUtf8());
    }

    QNetworkAccessManager *manager = new QNetworkAccessManager();
    QNetworkReply *reply = posted ? manager->post(request, params.toString(QUrl::FullyEncoded).toUtf8()) : manager->get(request);

    // 请求错误处理
    QObject::connect(reply, static_cast<void (QNetworkReply::*)(QNetworkReply::NetworkError)>(&QNetworkReply::error), [=] {
        if (NULL != errorHandler) {
            errorHandler(reply->errorString());
        }
    });

    // 请求结束时一次性读取所有响应数据
    QObject::connect(reply, &QNetworkReply::finished, [=] {
        if (reply->error() == QNetworkReply::NoError && NULL != successHandler) {
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
        manager->deleteLater();
    });
}
```

## 服务器端处理请求的代码
这里的服务器端处理请求的代码使用了 `SpringMVC` 实现，可以对应的换为其他实现。

```java
@GetMapping("/device")
@ResponseBody
public String detectDevice(Device device) {
    if (device.isMobile()) {
        return "Mobile";
    } else if (device.isTablet()) {
        return "Tablet";
    } else {
        return "Desktop";
    }
}
    
// URL: /signIn?id=1&name=xxx
@GetMapping("/signIn")
@ResponseBody
public String singInGet(@RequestParam String id,
                     @RequestParam String name,
                     @RequestHeader(value="token", required=false) String token) throws Exception {
    name = new String(name.getBytes("iso8859-1"), "UTF-8");
    return String.format("GET: id: %s, name: %s, token: %s", id, name, token);
}

// URL: /signIn
@PostMapping("/signIn")
@ResponseBody
public String singInPost(@RequestParam String id,
                     @RequestParam String name,
                     @RequestHeader(value="token", required=false) String token) throws Exception {
    return String.format("POST: id: %s, name: %s, token: %s", id, name, token);
}
```
