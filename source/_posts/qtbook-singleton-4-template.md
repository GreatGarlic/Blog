---
title: 单例的模版实现
date: 2016-12-15 19:31:42
tags: QtBook
---
相信现在大家对单例的实现已经成竹于胸，接下来，介绍使用模版技术实现单例。

## Singleton.h

对于不同的类型，只能使用类模版 `Singleton` 生成一个唯一的对象。

```cpp
#ifndef SINGLETON_H
#define SINGLETON_H

#include <QMutex>
#include <QScopedPointer>

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

/*-----------------------------------------------------------------------------|
 |                          Singleton implementation                           |
 |----------------------------------------------------------------------------*/
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

#endif // SINGLETON_H
```

## ConfigUtil.h

```cpp
#ifndef CONFIGUTIL_H
#define CONFIGUTIL_H
#include <QString>
#include <QScopedPointer> // [1]

template <typename T> class Singleton; // [2]

class ConfigUtil {
public:
    QString getDatabaseName() const;

private:
    // [3]
    ConfigUtil();
    ~ConfigUtil();
    ConfigUtil(const ConfigUtil &other);
    ConfigUtil& operator=(const ConfigUtil &other);

    // [4]
    friend class Singleton<ConfigUtil>;
    friend struct QScopedPointerDeleter<ConfigUtil>;
};

#endif // CONFIGUTIL_H
```

由于 ConfigUtil 的构造函数是 private 的，要在类 Singleton 里生成它的对象，所以需要把 Singleton 设置为它的友员类，同理由于它的析构函数是 private 的，程序退出是会在 QScopedPointerDeleter 里 delete 它的指针对象，所以要把 QScopedPointerDeleter 设置为它的友员结构体。

## ConfigUtil.cpp

```cpp
#include "ConfigUtil.h"
#include <QDebug>

ConfigUtil::ConfigUtil() {
    qDebug() << "ConfigUtil()";
}

ConfigUtil::~ConfigUtil() {
    qDebug() << "~ConfigUtil()";
}

QString ConfigUtil::getDatabaseName() const {
    return "The Mass";
}
```

## main.cpp
`main()` 函数用于展示 ConfigUtil 的使用。

```cpp
#include "ConfigUtil.h"
#include "Singleton.h"
#include <QDebug>

int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    qDebug() << Singleton<ConfigUtil>::getInstance().getDatabaseName();
    qDebug() << Singleton<ConfigUtil>::getInstance().getDatabaseName();

    return 0;
}
```
输出：
> ConfigUtil()  
> "The Mass"  
> "The Mass"  
> ~ConfigUtil()

单例的功能没有任何问题，唯一和前面不同的是，获取单例类的对象使用的是 `Singleton<ConfigUtil>::getInstance()` 而不是 ~~ConfigUtil::getInstance()~~。

## 思考
和前面一样，观察 ConfigUtil 的实现，也是同样的有大量相似的重复代码用来实现单例，在 ConfigUtil.h 里标记为 `[1]`，`[2]`，`[3]`，`[4]` 的部分，有了前面的经验，我们应该能轻易的想到使用宏实现代码复用，简化实现过程。具体怎么做呢？我们可以先自己独立地实现一下，然后再阅读后面的章节，相信对我们会更有启发意义。
