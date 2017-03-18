---
title: Single Application
date: 2016-11-15 14:44:01
tags: QtBook
---
如果限制一个程序同时只能启动一个实例，有几个可以使用的库

* QtSingleApplication

    > 以前可以免费使用，后来只有商业版能里能用，在 Github 上也有一个 LGPL 协议的实现，地址为 <https://github.com/qtproject/qt-solutions/tree/master/qtsingleapplication>

* SingleApplication

    > This is a replacement of the QSingleApplication for Qt5，地址为 <https://github.com/itay-grudev/SingleApplication>

* RunGuard

    > 使用共享内存、简单、轻量化的实现，缺点是程序间不能通信，双击程序图标的时候不会激活已运行的实例显示到最前面，下面介绍 RunGuard 的使用

<!--more-->

## main.cpp
限制程序同时只能启动一个实例只需要在 `main()` 函数里调用 `if (!guard.tryToRun()) { return 0; }` 即可。

```cpp
#include "RunGuard.h"

int main(int argc, char *argv[]) {
    RunGuard guard("9F0FFF1A-77A0-4EF0-87F4-5494CA8181C7");

    if (!guard.tryToRun()) {
        return 0;
    }

    QApplication a(argc, argv);
    a.exec();
}
```

## RunGuard.h
```cpp
#ifndef RUNGUARD_H
#define RUNGUARD_H

#include <QObject>
#include <QSharedMemory>
#include <QSystemSemaphore>

class RunGuard {
public:
    RunGuard(const QString& key);
    ~RunGuard();

    bool isAnotherRunning();
    bool tryToRun();
    void release();

private:
    const QString key;
    const QString memLockKey;
    const QString sharedmemKey;

    QSharedMemory sharedMem;
    QSystemSemaphore memLock;

    Q_DISABLE_COPY(RunGuard)
};
#endif // RUNGUARD_H
```

## RunGuard.cpp
```cpp
#include "RunGuard.h"
#include <QCryptographicHash>

namespace {

QString generateKeyHash(const QString& key, const QString& salt) {
    QByteArray data;

    data.append(key.toUtf8());
    data.append(salt.toUtf8());
    data = QCryptographicHash::hash(data, QCryptographicHash::Sha1).toHex();

    return data;
}

}


RunGuard::RunGuard(const QString &key)
    : key(key)
    , memLockKey(generateKeyHash(key, "_memLockKey"))
    , sharedmemKey(generateKeyHash(key, "_sharedmemKey"))
    , sharedMem(sharedmemKey)
    , memLock(memLockKey, 1) {
    memLock.acquire();
    {
        QSharedMemory fix(sharedmemKey); // Fix for *nix: http://habrahabr.ru/post/173281/
        fix.attach();
    }
    memLock.release();
}

RunGuard::~RunGuard() {
    release();
}

bool RunGuard::isAnotherRunning() {
    if (sharedMem.isAttached()) {
        return false;
    }

    memLock.acquire();
    const bool isRunning = sharedMem.attach();
    if (isRunning) {
        sharedMem.detach();
    }
    memLock.release();

    return isRunning;
}

bool RunGuard::tryToRun() {
    if (isAnotherRunning()) { // Extra check
        return false;
    }

    memLock.acquire();
    const bool result = sharedMem.create(sizeof(quint64));
    memLock.release();

    if (!result) {
        release();
        return false;
    }

    return true;
}

void RunGuard::release() {
    memLock.acquire();

    if (sharedMem.isAttached()) {
        sharedMem.detach();
    }

    memLock.release();
}
```
