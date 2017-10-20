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
更多的使用方法请参考 `main()` 里的例子。HttpClient 的实现为 `HttpClient.h` 和 `HttpClient.cpp` 部分。

## main.cpp
`main()` 函数里展示了 `HttpClient` 的使用示例。

```cpp
#include "HttpClient.h"

#include <QDebug>
#include <QFile>
#include <QApplication>
#include <QNetworkAccessManager>
#include <QThread>
#include <QDate>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    // 在代码块里执行网络访问，是为了测试 HttpClient 对象在被析构后，网络访问的回调函数仍然能正常执行
    {
        // [[1]] GET 请求无参数
        HttpClient("http://localhost:8080/device").get([](const QString &response) {
            qDebug() << response;
        });

        // [[2]] GET 请求有参数，有自定义 header
        HttpClient("http://localhost:8080/signIn")
                .addParam("id", "1")
                .addParam("name", "诸葛亮")
                .addHeader("token", "123AS#D")
                .get([](const QString &response) {
            qDebug() << response;
        });

        // [[3]] POST 请求有参数，有自定义 header
        HttpClient("http://localhost:8080/signIn")
                .addParam("id", "2")
                .addParam("name", "卧龙")
                .addHeader("token", "DER#2J7")
                // .addHeader("content-type", "application/x-www-form-urlencoded")
                .addFormHeader()
                .post([](const QString &response) {
            qDebug() << response;
        });

        // [[4]] 每创建一个 QNetworkAccessManager 对象都会创建一个线程，当频繁的访问网络时，为了节省线程资源，调用 useManager()
        // 使用共享的 QNetworkAccessManager，它不会被 HttpClient 删除。
        // 如果下面的代码不传入 QNetworkAccessManager，从任务管理器里可以看到创建了几千个线程。
        QNetworkAccessManager *manager = new QNetworkAccessManager();
        for (int i = 0; i < 5000; ++i) {
            HttpClient("http://localhost:8080/device").useManager(manager).get([=](const QString &response) {
                qDebug() << response << ", " << i;
            });
        }

        // [[5]] 下载: 下载直接保存到文件
        HttpClient("http://xtuer.github.io/img/dog.png").debug(true).download("/Users/Biao/Desktop/dog.png");

        // [[6]] 下载: 自己处理下载得到的字节数据
        QFile *file = new QFile("dog.png");
        if (file->open(QIODevice::WriteOnly)) {
            HttpClient("http://xtuer.github.io/img/dog.png").debug(true).download([=](const QByteArray &data) {
                file->write(data);
            }, [=] {
                file->flush();
                file->close();
                file->deleteLater();

                qDebug() << "Download file finished";
            });
        }

        // [[7]] 上传
        HttpClient("http://localhost:8080/webuploader").upload("/Users/Biao/Pictures/ade.jpg");
    }

    return a.exec();
}
```

## HttpClient.h

```cpp
#ifndef HTTPCLIENT_H
#define HTTPCLIENT_H

#include <functional>

class QString;
class QByteArray;
struct HttpClientPrivate;
class QNetworkReply;
class QNetworkAccessManager;

/**
 * @brief 对 QNetworkAccessManager 进行封装的 HTTP 访问客户端，可以进行 GET，POST，上传，下载请求。
 */
class HttpClient {
public:
    HttpClient(const QString &url);
    ~HttpClient();

    /**
     * @brief 每创建一个 QNetworkAccessManager 对象都会创建一个线程，当频繁的访问网络时，为了节省线程资源，
     *     可以使用传人的 QNetworkAccessManager，它不会被 HttpClient 删除。
     *     如果没有使用 useManager() 传入一个 QNetworkAccessManager，则 HttpClient 会自动的创建一个，并且在网络访问完成后删除它。
     * @param  manager QNetworkAccessManager 对象
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& useManager(QNetworkAccessManager *manager);

    /**
     * @brief  参数 debug 为 true 则使用 debug 模式，请求执行时输出请求的 URL 和参数等
     * @param  debug 是否启用调试模式
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& debug(bool debug);

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
     * @brief 添加 POST 表单使用的头信息，等价于 addHeader("content-type", "application/x-www-form-urlencoded")
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& addFormHeader();

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

    /**
     * @brief 使用 GET 进行下载，下载的文件保存到 destinationPath
     * @param destinationPath 下载的文件保存路径
     * @param finishHandler   请求处理完成后的回调 lambda 函数
     * @param errorHandler    请求失败的回调 lambda 函数，打开文件 destinationPath 出错也会调用此函数
     */
    void download(const QString &destinationPath,
                  std::function<void ()> finishHandler = NULL,
                  std::function<void (const QString &)> errorHandler = NULL);

    /**
     * @brief 使用 GET 进行下载，当有数据可读取时回调 readyRead(), 大多数情况下应该在 readyRead() 里把数据保存到文件
     * @param readyRead     有数据可读取时的回调 lambda 函数
     * @param finishHandler 请求处理完成后的回调 lambda 函数
     * @param errorHandler  请求失败的回调 lambda 函数
     */
    void download(std::function<void (const QByteArray &)> readyRead,
                  std::function<void ()> finishHandler = NULL,
                  std::function<void (const QString &)> errorHandler = NULL);

    /**
     * @brief 上传文件
     * @param path 要上传的文件的路径
     * @param successHandler 请求成功的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     * @param encoding       请求响应的编码
     */
    void upload(const QString &path, std::function<void (const QString &)> successHandler = NULL,
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

    /**
     * @brief 读取服务器响应的数据
     * @param reply 请求的 QNetworkReply 对象
     * @param encoding 请求响应的编码，默认使用 UTF-8
     * @return 服务器端响应的字符串
     */
    QString readResponse(QNetworkReply *reply, const char *encoding = "UTF-8");

    HttpClientPrivate *d;
};

#endif // HTTPCLIENT_H
```

## HttpClient.cpp
```cpp
#include "HttpClient.h"

#include <QDebug>
#include <QFile>
#include <QHash>
#include <QUrlQuery>
#include <QNetworkReply>
#include <QNetworkRequest>
#include <QNetworkAccessManager>
#include <QHttpPart>
#include <QHttpMultiPart>

struct HttpClientPrivate {
    HttpClientPrivate(const QString &url) : url(url), networkAccessManager(NULL), useInternalNetworkAccessManager(true), debug(false) {}

    QString url; // 请求的 URL
    QUrlQuery params; // 请求的参数
    QHash<QString, QString> headers; // 请求的头
    QNetworkAccessManager *networkAccessManager;
    bool useInternalNetworkAccessManager; // 是否使用内部的 QNetworkAccessManager
    bool debug;
};

// 注意: 不能在回调函数中使用 d，因为回调函数被调用时 HttpClient 对象很可能已经被释放掉了。
HttpClient::HttpClient(const QString &url) : d(new HttpClientPrivate(url)) {
    //    qDebug() << "HttpClient";
}

HttpClient::~HttpClient() {
    //    qDebug() << "~HttpClient";
    delete d;
}

HttpClient &HttpClient::useManager(QNetworkAccessManager *manager) {
    d->networkAccessManager = manager;
    d->useInternalNetworkAccessManager = false;
    return *this;
}

// 传入 debug 为 true 则使用 debug 模式，请求执行时输出请求的 URL 和参数等
HttpClient &HttpClient::debug(bool debug) {
    d->debug = debug;
    return *this;
}

// 增加参数
HttpClient &HttpClient::addParam(const QString &name, const QString &value) {
    d->params.addQueryItem(name, value);
    return *this;
}

// 增加访问头
HttpClient &HttpClient::addHeader(const QString &header, const QString &value) {
    d->headers[header] = value;
    return *this;
}

HttpClient &HttpClient::addFormHeader() {
    return addHeader("content-type", "application/x-www-form-urlencoded");
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

void HttpClient::download(const QString &destinationPath,
                          std::function<void ()> finishHandler,
                          std::function<void (const QString &)> errorHandler) {
    QFile *file = new QFile(destinationPath);
    bool debug = d->debug;

    if (file->open(QIODevice::WriteOnly)) {
        download([=](const QByteArray &data) {
            file->write(data);
        }, [=] {
            // 请求结束后释放文件对象.
            file->flush();
            file->close();
            file->deleteLater();

            // 不能用 d->debug，因为 d 以及被释放了
            if (debug) {
                qDebug() << QString("下载完成，保存到: %1").arg(destinationPath);
            }

            if (NULL != finishHandler) {
                finishHandler();
            }
        }, errorHandler);
    } else {
        // 打开文件出错
        if (debug) {
            qDebug() << QString("打开文件出错: %1").arg(destinationPath);
        }

        if (NULL != errorHandler) {
            errorHandler(QString("打开文件出错: %1").arg(destinationPath));
        }
    }
}

// 使用 GET 进行下载，当有数据可读取时回调 readyRead(), 大多数情况下应该在 readyRead() 里把数据保存到文件
void HttpClient::download(std::function<void (const QByteArray &)> readyRead,
                          std::function<void ()> finishHandler,
                          std::function<void (const QString &)> errorHandler) {
    if (d->debug) {
        QString params = d->params.toString();

        if (params.isEmpty()) {
            qDebug() << QString("URL: %1").arg(d->url);
        } else {
            qDebug() << QString("URL: %1?%2").arg(d->url).arg(params);
        }
    }

    // 如果是 GET 请求，并且参数不为空，则编码请求的参数，放到 URL 后面
    if (!d->params.isEmpty()) {
        d->url += "?" + d->params.toString(QUrl::FullyEncoded);
    }

    QUrl urlx(d->url);
    QNetworkRequest request(urlx);
    bool internal = d->useInternalNetworkAccessManager;
    QNetworkAccessManager *manager = internal ? new QNetworkAccessManager() : d->networkAccessManager;
    QNetworkReply *reply = manager->get(request);

    // 有数据可读取时回调 readyRead()
    QObject::connect(reply, &QNetworkReply::readyRead, [=] {
        readyRead(reply->readAll());
    });

    // 请求结束
    QObject::connect(reply, &QNetworkReply::finished, [=] {
        if (reply->error() == QNetworkReply::NoError && NULL != finishHandler) {
            finishHandler();
        }

        // 释放资源
        reply->deleteLater();
        if (internal) {
            manager->deleteLater();
        }
    });

    // 请求错误处理
    QObject::connect(reply, static_cast<void (QNetworkReply::*)(QNetworkReply::NetworkError)>(&QNetworkReply::error), [=] {
        if (NULL != errorHandler) {
            errorHandler(reply->errorString());
        }
    });
}

void HttpClient::upload(const QString &path,
                        std::function<void (const QString &)> successHandler,
                        std::function<void (const QString &)> errorHandler,
                        const char *encoding) {
    if (d->debug) {
        qDebug() << QString("URL: %1").arg(d->url);
    }

    QHttpMultiPart *multiPart = new QHttpMultiPart(QHttpMultiPart::FormDataType);

    QFile *file = new QFile(path);
    file->setParent(multiPart); // we cannot delete the file now, so delete it with the multiPart

    // 如果文件打开失败，则释放资源返回
    if(!file->open(QIODevice::ReadOnly)) {
        if (NULL != errorHandler) {
            errorHandler(QString("文件打开失败: %1").arg(file->errorString()));
            multiPart->deleteLater();
            return;
        }
    }

    // 表明是文件上传
    QString disposition = QString("form-data; name=\"file\"; filename=\"%1\"").arg(file->fileName());
    QHttpPart imagePart;
    imagePart.setHeader(QNetworkRequest::ContentDispositionHeader, QVariant(disposition));
    imagePart.setBodyDevice(file);
    multiPart->append(imagePart);

    bool internal = d->useInternalNetworkAccessManager;
    QNetworkRequest request(QUrl(d->url));
    QNetworkAccessManager *manager = internal ? new QNetworkAccessManager() : d->networkAccessManager;
    QNetworkReply *reply = manager->post(request, multiPart);
    multiPart->setParent(reply);

    // 请求结束时一次性读取所有响应数据
    QObject::connect(reply, &QNetworkReply::finished, [=] {
        if (reply->error() == QNetworkReply::NoError && NULL != successHandler) {
            successHandler(readResponse(reply, encoding)); // 成功执行
        }

        // 释放资源
        reply->deleteLater();
        if (internal) {
            manager->deleteLater();
        }
    });

    // 请求错误处理
    QObject::connect(reply, static_cast<void (QNetworkReply::*)(QNetworkReply::NetworkError)>(&QNetworkReply::error), [=] {
        if (NULL != errorHandler) {
            errorHandler(reply->errorString());
        }
    });
}

// 执行请求的辅助函数
void HttpClient::execute(bool posted,
                         std::function<void (const QString &)> successHandler,
                         std::function<void (const QString &)> errorHandler,
                         const char *encoding) {
    // 如果是 GET 请求，并且参数不为空，则编码请求的参数，放到 URL 后面
    if (!posted && !d->params.isEmpty()) {
        d->url += "?" + d->params.toString(QUrl::FullyEncoded);
    }

    if (d->debug) {
        qDebug() << QString("URL: %1?%2").arg(d->url).arg(d->params.toString());
    }

    QUrl urlx(d->url);
    QNetworkRequest request(urlx);

    // 把请求的头添加到 request 中
    QHashIterator<QString, QString> iter(d->headers);
    while (iter.hasNext()) {
        iter.next();
        request.setRawHeader(iter.key().toUtf8(), iter.value().toUtf8());
    }

    // 注意: 不能在 Lambda 表达式里使用 HttpClient 对象的成员数据，因其可能在网络访问未结束时就已经被析构掉了，
    // 所以如果要使用它的相关数据，定义一个局部变量来保存其数据，然后在 Lambda 表达式里访问这个局部变量

    // 如果不使用外部的 manager 则创建一个新的，在访问完成后会自动删除掉
    bool internal = d->useInternalNetworkAccessManager;
    QNetworkAccessManager *manager = internal ? new QNetworkAccessManager() : d->networkAccessManager;
    QNetworkReply *reply = posted ? manager->post(request, d->params.toString(QUrl::FullyEncoded).toUtf8()) : manager->get(request);

    // 请求结束时一次性读取所有响应数据
    QObject::connect(reply, &QNetworkReply::finished, [=] {
        if (reply->error() == QNetworkReply::NoError && NULL != successHandler) {
            successHandler(readResponse(reply, encoding)); // 成功执行
        }

        // 释放资源
        reply->deleteLater();
        if (internal) {
            manager->deleteLater();
        }
    });

    // 请求错误处理
    QObject::connect(reply, static_cast<void (QNetworkReply::*)(QNetworkReply::NetworkError)>(&QNetworkReply::error), [=] {
        if (NULL != errorHandler) {
            errorHandler(reply->errorString());
        }
    });
}

QString HttpClient::readResponse(QNetworkReply *reply, const char *encoding) {
    QTextStream in(reply);
    QString result;
    in.setCodec(encoding);

    while (!in.atEnd()) {
        result += in.readLine();
    }

    return result;
}
```

## 服务器端处理请求的代码
这里的服务器端处理请求的代码使用了 `SpringMVC` 实现，作为参考，可以使用其他语言实现，例如 PHP，C#。

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

// 上传
@PostMapping("/upload")
@ResponseBody
public Result uploadFile(@RequestParam("file") MultipartFile file) throws IOException {
    System.out.println(file.getOriginalFilename());
    file.transferTo(new File("/Users/Biao/Desktop/" + file.getOriginalFilename()));

    return Result.ok("OK", file.getOriginalFilename());
}
```

