---
title: Qt 自定义日志工具
date: 2016-09-25 16:31:43
tags: Qt
---

C++ 中比较不错的日志工具有 `log4cxx`，`log4qt` 等，但是它们都不能和 `qDebug()`, `qInfo()` 等有机的结合在一起，所以在 Qt 中使用总觉得不够舒服，感谢 Qt 提供了 `qInstallMessageHandler()` 这个函数，使用这个函数可以安装自定义的日志输出处理函数，把日志输出到文件，控制台等，具体的使用可以查看 Qt 的帮助文档。

<!--more-->

本文主要是介绍使用 `qInstallMessageHandler()` 实现一个简单的日志工具，例如调用 `qDebug() << "Hi"`，输出的内容会同时输出到日志文件和控制台，并且日志文件如果不是当天创建的，会使用它的创建日期备份起来，涉及到的文件有:

* main.cpp: 使用示例
* Singleton.h: 单例模版
* LogHandler.h: 自定义日志相关类的头文件
* LogHandler.cpp: 自定义日志相关类的实现文件

## main.cpp
```cpp
#include "LogHandler.h"

#include <QApplication>
#include <QDebug>
#include <QTime>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    // [[1]] 安装消息处理函数
    Singleton<LogHandler>::getInstance().installMessageHandler();

    // [[2]] 输出测试，查看是否写入到文件
    qDebug() << "Hello";
    qDebug() << "当前时间是: " << QTime::currentTime().toString("hh:mm:ss");
    qInfo() << QString("God bless you!");

    QPushButton *button = new QPushButton("退出");
    button->show();
    QObject::connect(button, &QPushButton::clicked, [&app] {
        qDebug() << "退出";
        app.quit();
    });

    // [[3]] 删除自定义消息处理，然后启用
    Singleton<LogHandler>::getInstance().release();
    qDebug() << "........"; // 不写入日志
    Singleton<LogHandler>::getInstance().installMessageHandler();

    int ret = app.exec(); // 事件循环结束

    // [[4]] 程序结束时释放 LogHandler 的资源，例如刷新并关闭日志文件
    Singleton<LogHandler>::getInstance().release();

    return ret;
}
```

控制台输出:

```
Hello
当前时间是:  "16:29:42"
"God bless you!"
........
退出
```

日志文件:  
位置: exe 所在目录的 log 目录下的 log.txt  
格式: 时间 - \[Level] (文件名:行数, 函数): 消息

```
16:29:42 - [Debug] (main.cpp:15, int main(int, char **)): Hello
16:29:42 - [Debug] (main.cpp:16, int main(int, char **)): 当前时间是:  "16:29:42"
16:29:42 - [Info ] (main.cpp:17, int main(int, char **)): "God bless you!"
16:29:46 - [Debug] (main.cpp:22, auto main(int, char **)::(anonymous class)::operator()() const): 退出
```

## LogHandler.h
```cpp
#ifndef LOGHANDLER_H
#define LOGHANDLER_H

#include "Singleton.h"

struct LogHandlerPrivate;

class LogHandler {
    SINGLETON(LogHandler) // 使用单例模式
public:
    void release(); // 释放资源
    void installMessageHandler(); // 给 Qt 安装消息处理函数

private:
    LogHandlerPrivate *d;
};

#endif // LOGHANDLER_H
```

## LogHandler.cpp
```cpp
#include "LogHandler.h"

#include <stdio.h>
#include <stdlib.h>
#include <QDebug>
#include <QDateTime>
#include <QMutexLocker>
#include <QtGlobal>
#include <QDir>
#include <QFile>
#include <QFileInfo>
#include <QTimer>
#include <QTextStream>
#include <iostream>

/************************************************************************************************************
 *                                                                                                          *
 *                                               LogHandlerPrivate                                          *
 *                                                                                                          *
 ***********************************************************************************************************/
struct LogHandlerPrivate {
    LogHandlerPrivate();
    ~LogHandlerPrivate();

    // 打开日志文件 log.txt，如果日志文件不是当天创建的，则使用创建日期把其重命名为 yyyy-MM-dd.log，并重新创建一个 log.txt
    void openAndBackupLogFile();

    // 消息处理函数
    static void messageHandler(QtMsgType type, const QMessageLogContext &context, const QString &msg);

    // 如果日志所在目录不存在，则创建
    void makeSureLogDirectory() const;

    QDir   logDir;              // 日志文件夹
    QTimer renameLogFileTimer;  // 重命名日志文件使用的定时器
    QTimer flushLogFileTimer;   // 刷新输出到日志文件的定时器
    QDate  logFileCreatedDate;  // 日志文件创建的时间

    static QFile *logFile;      // 日志文件
    static QTextStream *logOut; // 输出日志的 QTextStream，使用静态对象就是为了减少函数调用的开销
    static QMutex logMutex;     // 同步使用的 mutex
};

// 初始化 static 变量
QMutex LogHandlerPrivate::logMutex;
QFile* LogHandlerPrivate::logFile = NULL;
QTextStream* LogHandlerPrivate::logOut = NULL;

LogHandlerPrivate::LogHandlerPrivate() {
    logDir.setPath("log"); // TODO: 日志文件夹的路径，为 exe 所在目录下的 log 文件夹，可从配置文件读取
    QString logPath = logDir.absoluteFilePath("log.txt"); // 日志的路径
    // 日志文件创建的时间
    // QFileInfo::created(): On most Unix systems, this function returns the time of the last status change.
    // 所以不能运行时使用这个函数检查创建时间，因为会在运行时变化，所以在程序启动时保存下日志文件创建的时间
    logFileCreatedDate = QFileInfo(logPath).created().date();

    // 打开日志文件，如果不是当天创建的，备份已有日志文件
    openAndBackupLogFile();

    // 十分钟检查一次日志文件创建时间
    renameLogFileTimer.setInterval(1000 * 60 * 10); // TODO: 可从配置文件读取
    // renameLogFileTimer.setInterval(1000); // 为了快速测试看到日期变化后是否新创建了对应的日志文件，所以 1 秒检查一次
    renameLogFileTimer.start();
    QObject::connect(&renameLogFileTimer, &QTimer::timeout, [this] {
        QMutexLocker locker(&LogHandlerPrivate::logMutex);
        openAndBackupLogFile();
    });

    // 定时刷新日志输出到文件，尽快的能在日志文件里看到最新的日志
    flushLogFileTimer.setInterval(1000); // TODO: 可从配置文件读取
    flushLogFileTimer.start();
    QObject::connect(&flushLogFileTimer, &QTimer::timeout, [this] {
        // qDebug() << QDateTime::currentDateTime().toString("yyyy-MM-dd hh:mm:ss"); // 测试不停的写入内容到日志文件
        QMutexLocker locker(&LogHandlerPrivate::logMutex);
        if (NULL != logOut) {
            logOut->flush();
        }
    });
}

LogHandlerPrivate::~LogHandlerPrivate() {
    if (NULL != logFile) {
        logFile->flush();
        logFile->close();
        delete logOut;
        delete logFile;

        // 因为他们是 static 变量
        logOut  = NULL;
        logFile = NULL;
    }
}

// 打开日志文件 log.txt，如果不是当天创建的，则使用创建日期把其重命名为 yyyy-MM-dd.log，并重新创建一个 log.txt
void LogHandlerPrivate::openAndBackupLogFile() {
    // 总体逻辑:
    // 1. 程序启动时 logFile 为 NULL，初始化 logFile，有可能是同一天打开已经存在的 logFile，所以使用 Append 模式
    // 2. logFileCreatedDate is null, 说明日志文件在程序开始时不存在，所以记录下创建时间
    // 3. 程序运行时检查如果 logFile 的创建日期和当前日期不相等，则使用它的创建日期重命名，然后再生成一个新的 log.txt 文件

    makeSureLogDirectory(); // 如果日志所在目录不存在，则创建
    QString logPath = logDir.absoluteFilePath("log.txt"); // 日志的路径

    // [[1]] 程序启动时 logFile 为 NULL
    if (NULL == logFile) {
        logFile = new QFile(logPath);
        logOut  = (logFile->open(QIODevice::WriteOnly | QIODevice::Text | QIODevice::Append)) ?  new QTextStream(logFile) : NULL;

        // [[2]] 如果文件是第一次创建，则创建日期是无效的，把其设置为当前日期
        if (logFileCreatedDate.isNull()) {
            logFileCreatedDate = QDate::currentDate();
        }

        // TODO: 可以检查日志文件超过 30 个，删除 30 天前的日志文件
    }

    // [[3]] 程序运行时如果创建日期不是当前日期，则使用创建日期重命名，并生成一个新的 log.txt
    if (logFileCreatedDate != QDate::currentDate()) {
        logFile->flush();
        logFile->close();
        delete logOut;
        delete logFile;

        QString newLogPath = logDir.absoluteFilePath(logFileCreatedDate.toString("yyyy-MM-dd.log"));;
        QFile::copy(logPath, newLogPath); // Bug: 按理说 rename 会更合适，但是 rename 时最后一个文件总是显示不出来，需要 killall Finder 后才出现
        QFile::remove(logPath); // 删除重新创建，改变创建时间

        logFile = new QFile(logPath);
        logOut  = (logFile->open(QIODevice::WriteOnly | QIODevice::Text | QIODevice::Truncate)) ?  new QTextStream(logFile) : NULL;
        logFileCreatedDate = QDate::currentDate();
    }
}

// 如果日志所在目录不存在，则创建
void LogHandlerPrivate::makeSureLogDirectory() const {
    if (!logDir.exists()) {
        logDir.mkpath("."); // 可以递归的创建文件夹
    }
}

// 消息处理函数
void LogHandlerPrivate::messageHandler(QtMsgType type, const QMessageLogContext &context, const QString &msg) {
    QMutexLocker locker(&LogHandlerPrivate::logMutex);
    QString level;

    switch (type) {
    case QtDebugMsg:
        level = "Debug";
        break;
    case QtInfoMsg:
        level = "Info ";
        break;
    case QtWarningMsg:
        level = "Warn ";
        break;
    case QtCriticalMsg:
        level = "Error";
        break;
    case QtFatalMsg:
        level = "Fatal";
        break;
    default:;
    }

    // 输出到标准输出
    QByteArray localMsg = msg.toUtf8();
    fprintf(stderr, "%s\n", localMsg.constData());

    if (NULL == LogHandlerPrivate::logOut) {
        return;
    }

    // 输出到日志文件, 格式: 时间 - [Level] (文件名:行数, 函数): 消息
    QString fileName = context.file;
    int index = fileName.lastIndexOf('/');
    fileName = fileName.mid(index + 1);

    (*LogHandlerPrivate::logOut) << QString("%1 - [%2] (%3:%4, %5): %6\n")
                                    .arg(QDateTime::currentDateTime().toString("yyyy-MM-dd hh:mm:ss")).arg(level)
                                    .arg(fileName).arg(context.line).arg(context.function).arg(msg);
}

/************************************************************************************************************
 *                                                                                                          *
 *                                               LogHandler                                                 *
 *                                                                                                          *
 ***********************************************************************************************************/
LogHandler::LogHandler() : d(NULL) {
}

LogHandler::~LogHandler() {
}

void LogHandler::installMessageHandler() {
    QMutexLocker locker(&LogHandlerPrivate::logMutex);

    if (NULL == d) {
        d = new LogHandlerPrivate();
        qInstallMessageHandler(LogHandlerPrivate::messageHandler); // 给 Qt 安装自定义消息处理函数
    }
}

void LogHandler::release() {
    QMutexLocker locker(&LogHandlerPrivate::logMutex);
    qInstallMessageHandler(0);
    delete d;
    d = NULL;
}
```

## Singleton.h
```cpp
#ifndef SINGLETON_H
#define SINGLETON_H

#include <QMutex>
#include <QScopedPointer>

////////////////////////////////////////////////////////////////////////////////
///                                                                          ///
///                            Singleton signature                           ///
///                                                                          ///
////////////////////////////////////////////////////////////////////////////////
/**
 * 使用方法:
 * 1. 定义类为单例:
 *     class ConnectionPool {
 *         SINGLETON(ConnectionPool) // Here
 *     public:
 *
 * 2. 实现无参构造函数，析构函数
 * 3. 获取单例类的对象:
 *     Singleton<ConnectionPool>::getInstance();
 *     ConnectionPool &pool = Singleton<ConnectionPool>::getInstance();
 * 注意: 如果单例的类需要释放的资源和 Qt 底层的信号系统有关系，例如 QSettings，QSqlDatabase 等，
 *     需要在程序结束前手动释放(也就是在 main() 函数返回前调用释放资源的函数)，
 *     否则有可能在程序退出时报系统底层的信号错误，导致如 QSettings 的数据没有保存。
 */
template <typename T>
class Singleton {
public:
    static T& getInstance();

    Singleton(const Singleton &other);
    Singleton<T>& operator=(const Singleton &other);

private:
    static QMutex mutex;
    static QScopedPointer<T> instance;
};

////////////////////////////////////////////////////////////////////////////////
///                                                                          ///
///                            Singleton definition                          ///
///                                                                          ///
////////////////////////////////////////////////////////////////////////////////
template <typename T> QMutex Singleton<T>::mutex;
template <typename T> QScopedPointer<T> Singleton<T>::instance;

template <typename T>
T& Singleton<T>::getInstance() {
    if (instance.isNull()) {
        mutex.lock();
        if (instance.isNull()) {
            instance.reset(new T());
        }
        mutex.unlock();
    }

    return *instance.data();
}

////////////////////////////////////////////////////////////////////////////////
///                                                                          ///
///                               Singleton Macro                            ///
///                                                                          ///
////////////////////////////////////////////////////////////////////////////////
#define SINGLETON(Class)                        \
private:                                        \
    Class();                                    \
    ~Class();                                   \
    Class(const Class &other);                  \
    Class& operator=(const Class &other);       \
    friend class Singleton<Class>;              \
    friend struct QScopedPointerDeleter<Class>;

#endif // SINGLETON_H
```

## 思考
1. main() 函数里的 qDebug() 输出都是在 UI 线程，LogHandler 是否多线程安全？怎么测试？
2. 日志的相关配置数据例如输出目录等都是写死在程序里的，如果写到配置文件里是不是更灵活？
3. 日志的格式也是写死在程序里的，如果能做到通过配置修改日志格式那就更强大了，就像 `log4cxx` 一样
4. 测试如何快速的看到不同日期生成的日志文件不同？
5. 删除超过 30 天的日志
6. 单个日志文件例如大于 100M 后重新创建一个新的日志文件
