---
title: 单例的智能指针＋宏的实现
date: 2016-12-15 19:19:15
tags: QtBook
---
如果要创建一个单例的数据库连接池 ConnectionPool，那么实现单例部分的代码和 ConfigUtil 的几乎一样，声明 private 的构造函数，拷贝构造函数，析构函数，赋值操作符，QScopedPointer instance，friend struct QScopedPointerDeleter，几乎完全一样的 getInstance() 等，这些代码几乎都是重复的，每个单例的类这些内容都重复一遍，违背了代码的复用原则。为了实现代码复用，可以用继承、函数、宏定义、模版等。对于单例，继承很难达到目的，这里我们选择使用宏来实现代码复用的目的，后面的章节会介绍使用模版的实现。<!--more-->

## Singleton.h

把实现单例可复用的代码抽取出来定义为宏放在文件 `Singleton.h` 中。

```cpp
#ifndef SINGLETON_H
#define SINGLETON_H

#include <QMutex>
#include <QScopedPointer>

// [1] 单例的主要内容声明与实现.
// Start of SINGLETON Declaration, Class 是类名
#define SINGLETON(Class)                        \
public:                                         \
    static Class& getInstance() {               \
        if (instance.isNull()) {                \
            mutex.lock();                       \
            if (instance.isNull()) {            \
                instance.reset(new Class());    \
            }                                   \
            mutex.unlock();                     \
        }                                       \
                                                \
        return *instance.data();                \
    }                                           \
                                                \
private:                                        \
    Class();                                    \
    ~Class();                                   \
    Class(const Class &other);                  \
    Class& operator=(const Class &other);       \
    static QMutex mutex;                        \
    static QScopedPointer<Class> instance;      \
    friend class QScopedPointer<Class>;         \
    friend struct QScopedPointerDeleter<Class>;

// End of SINGLETON Declaration

// [2] 静态对象的初始化
#define SINGLETON_STATIC_INITIALIZE(Class)      \
    QMutex Class::mutex;                        \
    QScopedPointer<Class> Class::instance;

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

SINGLETON_STATIC_INITIALIZE(ConfigUtil) // [3]

#include <QDebug>

ConfigUtil::ConfigUtil() {
    qDebug() << "ConfigUtil()";
}

ConfigUtil::~ConfigUtil() {
    qDebug() << "~ConfigUtil()";
}

QString ConfigUtil::getDatabaseName() const {
    return "Pandora";
}
```

##### 现在实现单例的类，单例相关的代码，只需要：

* [1] 包含头文件 `Singleton.h`
* [2] 使用宏 `SINGLETON`
* [3] 使用宏 `SINGLETON_STATIC_INITIALIZE`

重复的模版代码都定义在宏里，编译的时候宏自动展开生成相应的代码，和没有使用宏的时候代码是完全一样的。

接下来实现一个单例的 ConnectionPool 是不是很简单了呢？具体代码就不写出来了，就当作给大家的作业，注意参考 ConfigUtil 里的标记为 `[1]`，`[2]` 和 `[3]` 的地方。如果系统里有几十上百个类需要定义为单例的模式，使用宏是不是就非常方便了？

## main.cpp
`main()` 函数用于展示 ConfigUtil 的使用。

```cpp
#include "ConfigUtil.h"
#include <QDebug>

int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    qDebug() << ConfigUtil::getInstance().getDatabaseName();
    qDebug() << ConfigUtil::getInstance().getDatabaseName();

    return 0;
}
```

输出:
> ConfigUtil()  
> "Pandora"  
> "Pandora"  
> ~ConfigUtil()

没有任何问题，这下心里有底了吧。
