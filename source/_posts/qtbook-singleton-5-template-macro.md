---
title: 单例的模版＋宏的实现
date: 2016-12-15 19:42:35
tags: QtBook
---
下面，对 `单例的模版实现` 使用宏进一步简化。

## Singleton.h
在 Singleton.h 的最后面添加宏 `SINGLETON`

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

/*-----------------------------------------------------------------------------|
 |                               Singleton Macro                               |
 |----------------------------------------------------------------------------*/
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

## ConfigUtil.h

```cpp
#ifndef CONFIGUTIL_H
#define CONFIGUTIL_H

#include "Singleton.h" // [1]
#include <QString>

class ConfigUtil {
    SINGLETON(ConfigUtil) // [2]

public:
    QString getDatabaseName() const;
};

#endif // CONFIGUTIL_H
```

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
    return "Pirate";
}
```

##### 现在实现单例的类，单例相关的代码，只需要：

* [1] 包含头文件 `Singleton.h`
* [2] 使用宏 `SINGLETON`

在 `单例的智能指针＋宏的实现` 里用了两个宏，这里只用了一个，也算是个进步吧。

## main.cpp

`main()` 函数用于展示 ConfigUtil 的使用。

```cpp
#include "ConfigUtil.h"
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
> "Pirate"  
> "Pirate"  
> ~ConfigUtil()

通过上面对代码的优化，使用模版和宏，实现了大量代码的复用，只需要简单的几条语句，就能实现一个单例的类，让我们把更多的精力放在类的业务逻辑上，而不是浪费在单例实现的那些模版代码上。

如果觉得 `Singleton<ConfigUtil>::getInstance()` 还是太长，可以 里定义一个宏来简化访问:

```
// ConfigUtil.h
#define ConfigUtilInstance Singleton<ConfigUtil>::getInstance()

// main()
qDebug() << ConfigUtilInstance.getDatabaseName();
```
