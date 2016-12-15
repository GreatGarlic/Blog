---
title: 单例的智能指针实现
date: 2016-12-15 18:41:23
tags: QtBook
---
前面提出了一个问题：可不可以不需要我们手动的调用 release() 函数，程序结束前自动的删除单例类的对象呢？答案是可以，使用智能指针可以达到这个目的，这里我们使用的是 Qt 的 `QScopedPointer` 来实现，也可以使用标准的 C++ 的智能指针。

Qt 的帮助文档里对 `QScopedPointer` 的描述是
> The QScopedPointer class stores a pointer to a dynamically allocated object, and deletes it upon destruction.

也就是说，QScopedPointer 持有一个指针，当 QScopedPointer 的变量被析构的时候（使用栈变量就是超出作用域的时候），就会自动的调用 delete 删除它持有的指针对象。我们在单例类 ConfigUtil 里定义了一个静态的 QScopedPointer 的变量 instance，其持有 ConfigUtil 类的指针对象，程序结束的时候会自动析构 instance 而析构单例类的指针对象，所以就不需要在程序结束的时候手动的调用 release() 函数了。

关于 QScopedPointer 更多的信息请参考 Qt 的帮助文档，里面有很多例子，对理解 QScopedPointer 很有帮助，可以如下图在 QtCreator 里搜索：
![](/img/qtbook/singleton/Singleton-QScopedPointer.png)

## ConfigUtil.h

```cpp
#ifndef CONFIGUTIL_H
#define CONFIGUTIL_H

#include <QString>
#include <QMutex>
#include <QScopedPointer>

class ConfigUtil {
public:
    QString getDatabaseName() const;

    static ConfigUtil& getInstance();
    static void release();

private:
    ConfigUtil();
    ~ConfigUtil();
    ConfigUtil(const ConfigUtil &other);
    ConfigUtil& operator=(const ConfigUtil &other);

    static QMutex mutex;
    static QScopedPointer<ConfigUtil> instance;
    friend struct QScopedPointerDeleter<ConfigUtil>;
};

#endif // CONFIGUTIL_H
```

`static QScopedPointer<ConfigUtil> instance` 创建的静态变量 instance 会在程序结束前析构（在main() 函数返回之后），调用 delete 删除它持有的 ConfigUtil 的指针对象。

也许你会觉得有点奇怪的是 `friend struct QScopedPointerDeleter<ConfigUtil>`，为什么是声明 QScopedPointerDeleter 为 friend struct，而不是声明 QScopedPointer 为 friend class?

这和 QScopedPointer 的实现有关，QScopedPointer 被析构时不是直接在它的析构函数里调用 delete 删除它持有的指针对象，而是使用它的 Cleanup Handler 来删除，默认是 QScopedPointerDeleter，在 QScopedPointer 的帮助里有详细的说明：
> Custom Cleanup Handlers
>
> Arrays as well as pointers that have been allocated with malloc must not be deleted using delete. QScopedPointer's second template parameter can be used for custom cleanup handlers.
>
> The following custom cleanup handlers exist:
>
> * QScopedPointerDeleter - the default, deletes the pointer using delete
> * QScopedPointerArrayDeleter - deletes the pointer using delete []. Use this handler for pointers that were allocated with new [].
> * QScopedPointerPodDeleter - deletes the pointer using free(). Use this handler for pointers that were allocated with malloc().
> * QScopedPointerDeleteLater - deletes a pointer by calling deleteLater() on it. Use this handler for pointers to QObject's that are actively participating in a QEventLoop.
>
> You can pass your own classes as handlers, provided that they have a public static function void cleanup(T *pointer).

## ConfigUtil.cpp

```cpp
#include "ConfigUtil.h"
#include <QDebug>

QMutex ConfigUtil::mutex;
QScopedPointer<ConfigUtil> ConfigUtil::instance;

ConfigUtil::ConfigUtil() {
    qDebug() << "ConfigUtil()";
}

ConfigUtil::~ConfigUtil() {
    qDebug() << "~ConfigUtil()";
}

QString ConfigUtil::getDatabaseName() const {
    return "Verbose";
}

ConfigUtil& ConfigUtil::getInstance() {
    if (instance.isNull()) {
        mutex.lock();
        if (instance.isNull()) {
            instance.reset(new ConfigUtil());
        }
        mutex.unlock();
    }

    return *instance.data();
}
```

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
> "Verbose"  
> "Verbose"  
> ~ConfigUtil()

不需要像前面那样手动调用 release()，ConfigUtil 被自动析构了。

## 思考
从 ConfigUtil 的实现，我们知道了怎么写一个单例类，现在需要另一个单例的类如 ConnectionPool（请自己实现，相关的功能函数可以随便返回点东西表示一下就可以了），比较一下它们的代码是否有什么相似部分，是不是单例的实现部分都要抄一遍 ConfigUtil 相似的代码？
