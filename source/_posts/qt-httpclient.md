---
title: Qt 访问网络的 HttpClient
date: 2016-09-17 19:51:56
tags: Qt
---

Qt 使用 `QNetworkAccessManager` 访问网络，这里对其进行了简单的封装，访问网络的代码可以简化为:

```cpp
HttpClient("http://localhost:8080/rest").get([](const QString &response) {
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

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    // 在代码块里执行网络访问，是为了测试 HttpClient 对象在被析构后，网络访问的回调函数仍然能正常执行
    {
        QString url("http://localhost:8080/rest");

        // [[1]] GET 请求无参数
        HttpClient(url).get([](const QString &response) {
            qDebug().noquote() << response;
        });

        // [[2]] GET 请求有参数，有自定义 header
        HttpClient(url).debug(true).param("name", "诸葛亮").header("token", "md5sum").get([](const QString &response) {
            qDebug().noquote() << response;
        });

        // [[3]] POST 请求，使用 param 添加参数，请求的参数使用 Form 格式
        HttpClient(url).debug(true).param("name", "卧龙")
                .post([](const QString &response) {
            qDebug().noquote() << response;
        });

        // [[4]] PUT 请求，使用 json 添加参数，请求的参数使用 Json 格式
        HttpClient(url).debug(true).json("{\"name\": \"孔明\"}").put([](const QString &response) {
            qDebug().noquote() << response;
        });

        // [[5]] DELETE 请求
        HttpClient(url).debug(true).remove([](const QString &response) {
            qDebug().noquote() << response;
        });

        // [[6]] 下载: 直接保存到文件
        HttpClient("http://xtuer.github.io/img/dog.png").debug(true).download("/Users/Biao/Desktop/dog-1.png");

        // [[7]] 下载: 自己处理下载得到的字节数据
        QFile *file = new QFile("/Users/Biao/Desktop/dog-2.png");
        if (file->open(QIODevice::WriteOnly)) {
            HttpClient("http://xtuer.github.io/img/dog.png").debug(true).download([=](const QByteArray &data) {
                file->write(data);
            }, [=](const QString &) {
                file->flush();
                file->close();
                file->deleteLater();

                qDebug().noquote() << "下载完成";
            });
        } else {
            file->deleteLater();
            file = NULL;
        }

        // [[8]] 上传
        HttpClient("http://localhost:8080/upload").debug(true).upload("/Users/Biao/Pictures/ade.jpg");

        // [[9]] 上传: 也能同时传参数
        HttpClient("http://localhost:8080/upload").debug(true)
                .param("username", "Alice").param("password", "Passw0rd")
                .upload("/Users/Biao/Pictures/ade.jpg");
    }

    {
        // [[8]] 共享 QNetworkAccessManager
        // 每创建一个 QNetworkAccessManager 对象都会创建一个线程，当频繁的访问网络时，为了节省线程资源，调用 manager()
        // 使用共享的 QNetworkAccessManager，它不会被 HttpClient 删除，需要我们自己不用的时候删除它。
        // 如果下面的代码不传入 QNetworkAccessManager，从任务管理器里可以看到创建了几千个线程。
        QNetworkAccessManager *manager = new QNetworkAccessManager();
        for (int i = 0; i < 5000; ++i) {
            HttpClient("http://localhost:8080/rest").manager(manager).get([=](const QString &response) {
                qDebug().noquote() << response << ", " << i;
            });
        }
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
class QNetworkRequest;
class QNetworkReply;
class QNetworkAccessManager;
class HttpClientPrivate;

/**
 * 对 QNetworkAccessManager 进行简单封装的 HTTP 访问客户端，简化 GET、POST、PUT、DELETE、上传、下载等操作。
 * 执行请求可调用 get(), post(), put(), remove(), download(), upload()。
 * 在执行请求前可调用 header() 设置请求头，参数使用 Form 表单的方式传递则调用 param()，如果参数使用 request body
 * 传递则调用 json() 设置参数(当然也可以不是 JSON 格式，使用 request body 的情况多数是 RESTful 时，大家都是用 JSON 格式，故命名为 json)。
 * 默认 HttpClient 会创建一个 QNetworkAccessManager，如果不想使用默认的，调用 manager() 传入即可。
 * 默认不输出请求的网址参数等调试信息，如果需要输出，调用 debug(true) 即可。
 */
class HttpClient {
public:
    HttpClient(const QString &url);
    ~HttpClient();

    /**
     * @brief 每创建一个 QNetworkAccessManager 对象都会创建一个线程，当频繁的访问网络时，为了节省线程资源，
     *     可以传入 QNetworkAccessManager 给多个请求共享(它不会被 HttpClient 删除，用户需要自己手动删除)。
     *     如果没有使用 useManager() 传入一个 QNetworkAccessManager，则 HttpClient 会自动的创建一个，并且在网络访问完成后删除它。
     * @param  manager QNetworkAccessManager 对象
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& manager(QNetworkAccessManager *manager);

    /**
     * @brief  参数 debug 为 true 则使用 debug 模式，请求执行时输出请求的 URL 和参数等
     * @param  debug 是否启用调试模式
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& debug(bool debug);

    /**
     * @brief 添加请求的参数
     * @param name  参数的名字
     * @param value 参数的值
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& param(const QString &name, const QString &value);

    /**
     * @brief 添加请求的参数，使用 Json 格式，例如 "{\"name\": \"Alice\"}"
     * @param json Json 格式的参数字符串
     * @return
     */
    HttpClient& json(const QString &json);

    /**
     * @brief 添加请求头
     * @param header 请求头的名字
     * @param value  请求头的值
     * @return 返回 HttpClient 的引用，可以用于链式调用
     */
    HttpClient& header(const QString &header, const QString &value);

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
     * @brief 执行 PUT 请求
     * @param successHandler 请求成功的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     * @param encoding       请求响应的编码
     */
    void put(std::function<void (const QString &)> successHandler,
             std::function<void (const QString &)> errorHandler = NULL,
             const char *encoding = "UTF-8");

    /**
     * @brief 执行 DELETE 请求
     *        由于 delete 是 C++ 的运算符，所以用同义词 remove
     * @param successHandler 请求成功的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     * @param encoding       请求响应的编码
     */
    void remove(std::function<void (const QString &)> successHandler,
             std::function<void (const QString &)> errorHandler = NULL,
             const char *encoding = "UTF-8");

    /**
     * @brief 使用 GET 进行下载，下载的文件保存到 savePath
     * @param savePath       下载的文件保存路径
     * @param successHandler 请求处理完成后的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数，打开文件 destinationPath 出错也会调用此函数
     */
    void download(const QString &savePath,
                  std::function<void (const QString &)> successHandler = NULL,
                  std::function<void (const QString &)> errorHandler = NULL);

    /**
     * @brief 使用 GET 进行下载，当有数据可读取时回调 readyRead(), 大多数情况下应该在 readyRead() 里把数据保存到文件
     * @param readyRead      有数据可读取时的回调 lambda 函数
     * @param successHandler 请求处理完成后的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     */
    void download(std::function<void (const QByteArray &)> readyRead,
                  std::function<void (const QString &)> successHandler = NULL,
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

class HttpClientPrivate {
public:
    HttpClientPrivate(const QString &url);

    QString   url;    // 请求的 URL
    QUrlQuery params; // 请求的参数使用 Form 格式
    QString   json;   // 请求的参数使用 Json 格式
    QHash<QString, QString> headers; // 请求头
    QNetworkAccessManager  *manager;

    bool useJson; // 为 true 时请求使用 Json 格式传递参数，否则使用 Form 格式传递参数
    bool debug;   // 为 true 时输出请求的 URL 和参数

    // HTTP 请求的类型
    enum HttpMethod {
        GET, POST, PUT, DELETE, UPLOAD /* 不是 HTTP Method，只是为了上传时特殊处理而定义的 */
    };

    /**
     * @brief 执行请求的辅助函数
     * @param method         请求的类型
     * @param d              HttpClient 的辅助对象
     * @param successHandler 请求成功的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     * @param encoding       请求响应的编码
     */
    static void executeQuery(HttpMethod method, HttpClientPrivate *d,
                             std::function<void (const QString &)> successHandler,
                             std::function<void (const QString &)> errorHandler,
                             const char *encoding);

    /**
     * @brief 读取服务器响应的数据
     * @param reply 请求的 QNetworkReply 对象
     * @param encoding 请求响应的编码，默认使用 UTF-8
     * @return 服务器端响应的字符串
     */
    static QString readReply(QNetworkReply *reply, const char *encoding = "UTF-8");

    /**
     * @brief 使用用户设定的 URL、请求头等创建 Request
     * @param method 请求的类型
     * @param d      HttpClientPrivate 的对象
     * @return 返回可用于执行请求的 QNetworkRequest
     */
    static QNetworkRequest createRequest(HttpMethod method, HttpClientPrivate *d);

    /**
     * @brief 请求结束的处理函数
     * @param debug          如果为 true 则输出调试信息，为 false 不输出
     * @param successMessage 请求成功的消息
     * @param errorMessage   请求失败的消息
     * @param successHandler 请求成功的回调 lambda 函数
     * @param errorHandler   请求失败的回调 lambda 函数
     * @param reply          QNetworkReply 对象，不能为 NULL
     * @param manager        请求的 manager，不为 NULL 时在此函数中 delete
     */
    static void handleFinish(bool debug,
                             const QString &successMessage,
                             const QString &errorMessage,
                             std::function<void (const QString &)> successHandler,
                             std::function<void (const QString &)> errorHandler,
                             QNetworkReply *reply, QNetworkAccessManager *manager);
};

HttpClientPrivate::HttpClientPrivate(const QString &url) : url(url), manager(NULL), useJson(false), debug(false) {
}

// 注意: 不要在回调函数中使用 d，因为回调函数被调用时 HttpClient 对象很可能已经被释放掉了。
HttpClient::HttpClient(const QString &url) : d(new HttpClientPrivate(url)) {
}

HttpClient::~HttpClient() {
    delete d;
}

HttpClient &HttpClient::manager(QNetworkAccessManager *manager) {
    d->manager = manager;
    return *this;
}

// 传入 debug 为 true 则使用 debug 模式，请求执行时输出请求的 URL 和参数等
HttpClient &HttpClient::debug(bool debug) {
    d->debug = debug;

    return *this;
}

// 添加 Form 格式参数
HttpClient &HttpClient::param(const QString &name, const QString &value) {
    d->params.addQueryItem(name, value);

    return *this;
}

// 添加 Json 格式参数
HttpClient &HttpClient::json(const QString &json) {
    d->useJson  = true;
    d->json = json;

    return *this;
}

// 添加访问头
HttpClient &HttpClient::header(const QString &header, const QString &value) {
    d->headers[header] = value;

    return *this;
}

// 执行 GET 请求
void HttpClient::get(std::function<void (const QString &)> successHandler,
                     std::function<void (const QString &)> errorHandler,
                     const char *encoding) {
    HttpClientPrivate::executeQuery(HttpClientPrivate::GET, d, successHandler, errorHandler, encoding);
}

// 执行 POST 请求
void HttpClient::post(std::function<void (const QString &)> successHandler,
                      std::function<void (const QString &)> errorHandler,
                      const char *encoding) {
    HttpClientPrivate::executeQuery(HttpClientPrivate::POST, d, successHandler, errorHandler, encoding);
}

// 执行 PUT 请求
void HttpClient::put(std::function<void (const QString &)> successHandler,
                     std::function<void (const QString &)> errorHandler,
                     const char *encoding) {
    HttpClientPrivate::executeQuery(HttpClientPrivate::PUT, d, successHandler, errorHandler, encoding);
}

// 执行 DELETE 请求
void HttpClient::remove(std::function<void (const QString &)> successHandler,
                        std::function<void (const QString &)> errorHandler,
                        const char *encoding) {
    HttpClientPrivate::executeQuery(HttpClientPrivate::DELETE, d, successHandler, errorHandler, encoding);
}

void HttpClient::download(const QString &destinationPath,
                          std::function<void (const QString &)> successHandler,
                          std::function<void (const QString &)> errorHandler) {
    bool debug  = d->debug;
    QFile *file = new QFile(destinationPath);

    if (file->open(QIODevice::WriteOnly)) {
        download([=](const QByteArray &data) {
            file->write(data);
        }, [=](const QString &) {
            // 请求结束后释放文件对象
            file->flush();
            file->close();
            file->deleteLater();

            if (debug) {
                qDebug().noquote() << QString("下载完成，保存到: %1").arg(destinationPath);
            }

            if (NULL != successHandler) {
                successHandler(QString("下载完成，保存到: %1").arg(destinationPath));
            }
        }, errorHandler);
    } else {
        // 打开文件出错
        if (debug) {
            qDebug().noquote() << QString("打开文件出错: %1").arg(destinationPath);
        }

        if (NULL != errorHandler) {
            errorHandler(QString("打开文件出错: %1").arg(destinationPath));
        }
    }
}

// 使用 GET 进行下载，当有数据可读取时回调 readyRead(), 大多数情况下应该在 readyRead() 里把数据保存到文件
void HttpClient::download(std::function<void (const QByteArray &)> readyRead,
                          std::function<void (const QString &)> successHandler,
                          std::function<void (const QString &)> errorHandler) {
    bool debug    = d->debug;
    bool internal = d->manager == NULL;
    QNetworkRequest request        = HttpClientPrivate::createRequest(HttpClientPrivate::GET, d);
    QNetworkAccessManager *manager = internal ? new QNetworkAccessManager() : d->manager;
    QNetworkReply *reply           = manager->get(request);

    // 有数据可读取时回调 readyRead()
    QObject::connect(reply, &QNetworkReply::readyRead, [=] {
        readyRead(reply->readAll());
    });

    // 请求结束
    QObject::connect(reply, &QNetworkReply::finished, [=] {
        QString successMessage = "下载完成"; // 请求结束时一次性读取所有响应数据
        QString errorMessage   = reply->errorString();
        HttpClientPrivate::handleFinish(debug, successMessage, errorMessage, successHandler, errorHandler,
                                        reply, internal ? manager : NULL);
    });
}

void HttpClient::upload(const QString &path,
                        std::function<void (const QString &)> successHandler,
                        std::function<void (const QString &)> errorHandler,
                        const char *encoding) {
    bool debug = d->debug;
    QHttpMultiPart *multiPart = new QHttpMultiPart(QHttpMultiPart::FormDataType);

    // 创建 Form 表单的参数 Text Part
    QList<QPair<QString, QString> > paramItems = d->params.queryItems();
    for (int i = 0; i < paramItems.size(); ++i) {
        QHttpPart textPart;
        QString name  = paramItems.at(i).first;
        QString value = paramItems.at(i).second;
        textPart.setHeader(QNetworkRequest::ContentDispositionHeader, QString("form-data; name=\"%1\"").arg(name));
        textPart.setBody(value.toUtf8());
        multiPart->append(textPart);
    }

    // 创建 File Part
    QFile *file = new QFile(path);
    file->setParent(multiPart); // we cannot delete the file now, so delete it with the multiPart

    // 如果文件打开失败，则释放资源返回
    if(!file->open(QIODevice::ReadOnly)) {
        QString errorMessage = QString("打开文件失败[%2]: %1").arg(path).arg(file->errorString());

        if (debug) {
            qDebug().noquote() << errorMessage;
        }

        if (NULL != errorHandler) {
            errorHandler(errorMessage);
        }

        multiPart->deleteLater();
        return;
    }

    // 文件上传的参数名为 file，值为文件名
    QString disposition = QString("form-data; name=\"file\"; filename=\"%1\"").arg(file->fileName());
    QHttpPart filePart;
    filePart.setHeader(QNetworkRequest::ContentDispositionHeader, QVariant(disposition));
    filePart.setBodyDevice(file);
    multiPart->append(filePart);

    bool internal = d->manager == NULL;
    QNetworkRequest request        = HttpClientPrivate::createRequest(HttpClientPrivate::UPLOAD, d);
    QNetworkAccessManager *manager = internal ? new QNetworkAccessManager() : d->manager;
    QNetworkReply *reply           = manager->post(request, multiPart);

    QObject::connect(reply, &QNetworkReply::finished, [=] {
        multiPart->deleteLater(); // 释放资源: multiPart + file

        QString successMessage = HttpClientPrivate::readReply(reply, encoding); // 请求结束时一次性读取所有响应数据
        QString errorMessage   = reply->errorString();
        HttpClientPrivate::handleFinish(debug, successMessage, errorMessage, successHandler, errorHandler,
                                        reply, internal ? manager : NULL);
    });
}

// 执行请求的辅助函数
void HttpClientPrivate::executeQuery(HttpMethod method, HttpClientPrivate *d,
                                     std::function<void (const QString &)> successHandler,
                                     std::function<void (const QString &)> errorHandler,
                                     const char *encoding) {
    // 如果不使用外部的 manager 则创建一个新的，在访问完成后会自动删除掉
    bool debug    = d->debug;
    bool internal = d->manager == NULL;
    QNetworkRequest request        = createRequest(method, d);
    QNetworkAccessManager *manager = internal ? new QNetworkAccessManager() : d->manager;
    QNetworkReply *reply           = NULL;

    switch (method) {
    case HttpClientPrivate::GET:
        reply = manager->get(request);
        break;
    case HttpClientPrivate::POST:
        reply = manager->post(request, d->useJson ? d->json.toUtf8() : d->params.toString(QUrl::FullyEncoded).toUtf8());
        break;
    case HttpClientPrivate::PUT:
        reply = manager->put(request, d->useJson ? d->json.toUtf8() : d->params.toString(QUrl::FullyEncoded).toUtf8());
        break;
    case HttpClientPrivate::DELETE:
        reply = manager->deleteResource(request);
        break;
    }

    QObject::connect(reply, &QNetworkReply::finished, [=] {
        QString successMessage = HttpClientPrivate::readReply(reply, encoding); // 请求结束时一次性读取所有响应数据
        QString errorMessage   = reply->errorString();
        HttpClientPrivate::handleFinish(debug, successMessage, errorMessage, successHandler, errorHandler,
                                        reply, internal ? manager : NULL);
    });
}

QString HttpClientPrivate::readReply(QNetworkReply *reply, const char *encoding) {
    QTextStream in(reply);
    QString result;
    in.setCodec(encoding);

    while (!in.atEnd()) {
        result += in.readLine();
    }

    return result;
}

QNetworkRequest HttpClientPrivate::createRequest(HttpMethod method, HttpClientPrivate *d) {
    bool get      = method == HttpMethod::GET;
    bool upload   = method == HttpClientPrivate::UPLOAD;
    bool postForm = !get && !upload && !d->useJson;
    bool postJson = !get && !upload &&  d->useJson;

    // 如果是 GET 请求，并且参数不为空，则编码请求的参数，放到 URL 后面
    if (get && !d->params.isEmpty()) {
        d->url += "?" + d->params.toString(QUrl::FullyEncoded);
    }

    // 调试时输出网址和参数
    if (d->debug) {
        qDebug().noquote() << "网址:" << d->url;

        if (postJson) {
            qDebug().noquote() << "参数:" << d->json;
        } else if (postForm) {
            qDebug().noquote() << "参数:" << d->params.toString();
        } else if (upload) {
            qDebug().noquote() << "参数:" << d->params.toString();
        }
    }

    // 如果是 POST 请求，useJson 为 true 时添加 Json 的请求头，useJson 为 false 时添加 Form 的请求头
    if (postForm) {
        d->headers["Content-Type"] = "application/x-www-form-urlencoded";
    } else if (postJson) {
        d->headers["Accept"]       = "application/json; charset=utf-8";
        d->headers["Content-Type"] = "application/json";
    }

    // 把请求的头添加到 request 中
    QNetworkRequest request(QUrl(d->url));
    QHashIterator<QString, QString> iter(d->headers);
    while (iter.hasNext()) {
        iter.next();
        request.setRawHeader(iter.key().toUtf8(), iter.value().toUtf8());
    }

    return request;
}

void HttpClientPrivate::handleFinish(bool debug,
                                     const QString &successMessage,
                                     const QString &errorMessage,
                                     std::function<void (const QString &)> successHandler,
                                     std::function<void (const QString &)> errorHandler,
                                     QNetworkReply *reply, QNetworkAccessManager *manager) {
    if (reply->error() == QNetworkReply::NoError) {
        // 请求成功
        if (debug) {
            qDebug().noquote() << QString("[成功]请求结束: %1").arg(successMessage);
        }

        if (NULL != successHandler) {
            successHandler(successMessage);
        }
    } else {
        // 请求失败
        if (debug) {
            qDebug().noquote() << QString("[失败]请求结束: %1").arg(errorMessage);
        }

        if (NULL != errorHandler) {
            errorHandler(errorMessage);
        }
    }

    // 释放资源
    reply->deleteLater();

    if (NULL != manager) {
        manager->deleteLater();
    }
}
```

## 服务器端处理请求的代码
这里的服务器端处理请求的代码使用了 `SpringMVC` 实现，作为参考，可以使用其他语言实现，例如 PHP，C#。

```java
import com.xtuer.bean.Result;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.util.Map;

@Controller
public class Controller {
    @GetMapping("/rest")
    @ResponseBody
    public Result get(@RequestParam(required = false) String name) {
        return Result.ok("GET", name);
    }

    @PostMapping("/rest")
    @ResponseBody
    public Result post(@RequestParam String name) {
        return Result.ok("POST", name);
    }

    @PutMapping("/rest")
    @ResponseBody
    public Result put(@RequestBody String name) {
        return Result.ok("PUT", name);
    }

    @DeleteMapping("/rest")
    @ResponseBody
    public Result delete() {
        return Result.ok("DELETE");
    }

    @PostMapping("/upload")
    @ResponseBody
    public Result uploadFile(@RequestParam("file") MultipartFile file,
                             @RequestParam(required = false) String username,
                             @RequestParam(required = false) String password) throws IOException {
        System.out.println("Username: " + username + ", Password: " + password);
        System.out.println(file.getOriginalFilename());
        file.transferTo(new File("/Users/Biao/Desktop/" + System.currentTimeMillis() + "-" + file.getOriginalFilename()));

        return Result.ok("OK", file.getOriginalFilename());
    }
}
```

