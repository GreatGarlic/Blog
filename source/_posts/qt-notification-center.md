---
title: 观察者模式的 NotificationCenter
date: 2016-11-12 14:51:23
tags: Qt
---
Qt 的信号槽在解耦方面做的非常好，sender 和 receiver 不需要互相知道，但是在使用 QObject::connect() 建立信号槽连接的时候是必须要同时知道 sender 和 receiver 的，这在绝大多数时候都是非常好用的，但是如果只想发送通知后，对此通知感兴趣的监听者就能自动的收到通知，同时还不用 QObject::connect() 建立信号槽连接，就可以使用下面介绍的 `NotificationCenter` 来实现。此外，NotificationCenter 内部已经处理好了跨线程通讯，不需要再考虑不同线程间函数调用的头疼问题。

**NotificationCenter 的使用:**

* 通知的监听器，不需要使用继承，只需要实现函数 `notified`，当有通知的时候，`notified` 函数会被自动调用

    ```cpp
    Q_INVOKABLE void notified(int notificationId, const QByteArray &data);
    ```

* 监听某个通知，通知的标志是一个整数，可以根据业务随意定义

    ```cpp
    Singleton<NotificationCenter>::getInstance().addObserver(1, &foo);
    ```
* 发送通知, 只需要和 NotificationCenter 交互就能发送和接收通知

    ```cpp
    Singleton<NotificationCenter>::getInstance().notify(1, QString("Two").toUtf8()); // 同线程发送消息
    NOTIFY_CROSS_THREAD(2, QString("Two").toUtf8()); // 跨线程发送消息，也可同线程
    ```
* 删除监听器，不再需要的时候，从 NotificationCenter 删除监听器，以免造成野指针异常

    ```cpp
    Singleton<NotificationCenter>::getInstance().removeObserver(&foo);
    Singleton<NotificationCenter>::getInstance().removeObserver(1, &foo);
    ```

**NotificationCenter 什么时候使用? 以 QTcpSocket 通讯为例:**
使用信号槽和 NotificationCenter 都可以完成任务，但是 NotificationCenter 也是个不错的选择

* 如果每一种消息都对应一个信号，则一般都会对应一个槽函数(对创建太多函数总有莫名的恐惧感)
* 消息的种类会很多，就会需要太多的信号槽
* 需要使用 connect() 給信号槽对应的创建连接
* 如果要增加新的消息类型时，则要创建它的槽函数和建立连接
* 也许你会说，所有的消息都用同一个信号和槽函数，因为可以在发射时传入消息的种类，在槽函数里也是能区别消息的。是的，这确实是一个不错的方法，这种做法和我们的 NotificationCenter 其实就差不多一样了，但是有点区别的是它会把所有的消息都发送到槽函数里，不需要的也会发送，例如有 100 种消息，即使只对其中 2 种消息感兴趣，但是它也会收到另外的 98 种消息。 而使用 NotificationCenter 时只会收到你感兴趣的 2 种消息。
* 还有一点，NotificationCenter 中通知的监听器注册只需要关心通知的 ID，这个 ID 是根据业务需求定义的一个整数，不需要知道通知的发送者，但是建立信号槽连接时 sender 和 receiver 都需要知道，这也是相对方便的一点。

> 文中的 `监听器`，`观察者` 指的是同一个意思；`消息`，`通知` 也是同一个意思。

<!--more-->

## 文件说明
* NotificationCenter 实现的文件包含
    * NotificationCenter.h
    * NotificationCenter.cpp
    * Singleton.h
* 以下几个文件是用于测试使用的
    * main.cpp
    * FooObserver.h and FooObserver.cpp
    * BarObserver.h and BarObserver.cpp
    * NotifyThread.h and NotifyThread.cpp

## NotificationCenter.h and NotificationCenter.cpp
```cpp
#ifndef NOTIFICATIONCENTER_H
#define NOTIFICATIONCENTER_H

#include <QObject>
#include <QSet>
#include <QHash>
#include <QMetaObject>

#include "Singleton.h"

/**
 * 如果发送通知的线程不是主线程，因为 NotificationCenter 的对象是主线程创建的，则调用此宏发送，当然，同线程也可以使用这个宏来发送消息
 */
#define NOTIFY_CROSS_THREAD(notificationId, data) QMetaObject::invokeMethod(&Singleton<NotificationCenter>::getInstance(), \
    "notify", Q_ARG(int, notificationId), Q_ARG(QByteArray, data));

/**
 * @brief 使用观察者模式实现的通知中心，解耦观察者和被观察者，他们无须互相知道，Qt 的信号槽也是观察者模式，在 connect 的时候需要知道 sender 和 receiver，
 * 而通过通知中心发消息的话，通知发送方和接收方都只和通知中心交互: 观察者只需要监听某个通知，不需要知道通知是谁发送的，通知发送者也只管通过通知中心发送通知即可。
 *
 * 注意：观察者需要实现 SLOT 或者 Q_INVOKABLE 的函数 void notified(int notificationId, const QByteArray &data)，这个函数通把通知发送給对应的观察者。
 * 为了简单不再多加个继承，所以没有为观察者定义一个抽象类的接口，然后观察者去继承实现这个抽象类的方法。因为使用了 QMetaObject::invokeMethod() 来实现
 * 有通知到达的调用，而且这样做还有一个好处是可以解决跨线程调用的问题。
 *
 * 如果通知的观察者没有实现 notified() 函数，则程序运行时会在命令行输出错误: QMetaObject::invokeMethod: No such method Xxxx::notified(int,QByteArray)
 */
class NotificationCenter : public QObject {
    Q_OBJECT
    SINGLETON(NotificationCenter)
public:
    /**
     * @brief 給传入的 notificationId 添加观察者 observer
     *
     * @param notificationId 通知的 id
     * @param observer 观察者
     */
    Q_INVOKABLE void addObserver(int notificationId, QObject *observer);

    /**
     * @brief 删除 notificationId 的观察者 observer
     *
     * @param notificationId 通知的 id
     * @param observer 观察者
     */
    Q_INVOKABLE void removeObserver(int notificationId, QObject *observer);

    /**
     * @brief 删除所有通知的观察者 observer
     *
     * @param observer 观察者
     */
    Q_INVOKABLE void removeObserver(QObject *observer);

    /**
     * @brief 給观察者发送 notificationId 的通知，通知的内容为 QByteArray。
     * 使用 QByteArray 是为了便于利用 QDataStream 序列化和反序列化。
     *
     * @param notificationId 通知的 id
     * @param data 通知的内容
     */
    Q_INVOKABLE void notify(int notificationId, const QByteArray &data);

private:
    QHash<int, QSet<QObject*> > observers; // 某一个通知的观察者: key 是通知的 id，value 是订阅了这个通知的所有观察者
};

#endif // NOTIFICATIONCENTER_H
```

```cpp
#include "NotificationCenter.h"
#include <QList>

NotificationCenter::NotificationCenter() {}

NotificationCenter::~NotificationCenter() {}

// 添加观察者
void NotificationCenter::addObserver(int notificationId, QObject *observer) {
    if (NULL == observer) { return; }

    QSet<QObject*> &obs = observers[notificationId];
    obs.insert(observer);
}

// 删除观察者
void NotificationCenter::removeObserver(int notificationId, QObject *observer) {
    QSet<QObject*> &obs = observers[notificationId];
    obs.remove(observer);
}

// 删除观察者
void NotificationCenter::removeObserver(QObject *observer) {
    QList<int> notificationIds = observers.keys();

    foreach (int notificationId, notificationIds) {
        removeObserver(notificationId, observer);
    }
}

// 給观察者发送通知
void NotificationCenter::notify(int notificationId, const QByteArray &data) {
    QSet<QObject*> obs = observers[notificationId];

    foreach(QObject *observer, obs) {
        QMetaObject::invokeMethod(observer, "notified", Q_ARG(int, notificationId), Q_ARG(QByteArray, data)); // 使用 invokeMethod 是为可跨线程进行函数调用
    }
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
 * 类模版 Singleton 用于辅助实现单例模式。
 * 使用方法，以定义类 ConnectionPool 的单例实现为例:
 * 0. 包含单例的头文件:
 *    #include "Singleton.h"
 *
 * 1. 声明类要实现单例:
 *    class ConnectionPool {
 *        SINGLETON(ConnectionPool) // Here
 *    public:
 *
 * 2. 实现单例类默认的构造函数和析构函数
 *    ConnectionPool::ConnectionPool() {};
 *    ConnectionPool::~ConnectionPool() {};
 *
 * 3. 获取单例类的对象，就可以调用它的函数了:
 *    QSqlDatabase connection = Singleton<ConnectionPool>::getInstance().getConnection();
 *
 *    ConnectionPool &pool = Singleton<ConnectionPool>::getInstance();
 *    QSqlDatabase connection = pool.getConnection();
 *
 * 注意: 如果单例的类需要释放的资源和 Qt 底层的信号系统有关系，例如 QSettings，QSqlDatabase 等，
 *    需要在程序结束前手动释放(也就是在 main() 函数返回前调用释放资源的函数)，
 *    否则有可能在程序退出时报系统底层的信号错误，导致如 QSettings 的数据没有保存。
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

## main.cpp
```cpp
#include <QApplication>
#include <QDebug>

#include "NotificationCenter.h"
#include "FooObserver.h"
#include "BarObserver.h"
#include "Message.h"
#include "NotifyThread.h"

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    NotificationCenter &nc = Singleton<NotificationCenter>::getInstance();
    FooObserver foo; // 通知的观察者
    BarObserver bar; // 通知的观察者

    // [1] 测试添加观察者
    // 通知 1 有 2 个观察者 foo and bar
    // 通知 1 有 1 个观察者 bar
    qDebug() << "---------------- foo and bar 都能收到通知 1 和 2 ----------------";
    nc.addObserver(1, &foo);
    nc.addObserver(1, &bar);
    nc.addObserver(2, &bar);
    nc.notify(1, QString("One").toUtf8()); // foo and bar 都收到消息
    NOTIFY_CROSS_THREAD(2, QString("Two").toUtf8()); // bar 收到消息(使用宏发送通知)

    // [2] 测试删除观察者
    // 删除通知 1 的观察者 bar，它还剩下观察者 foo
    // 通知 2 有观察者 bar
    qDebug() << "---------------- bar 不再收到通知 1，但能收到通知 2 ----------------";
    nc.removeObserver(1, &bar); // nc.removeObserver(&bar) 则 bar 不在接收到任何通知
    nc.notify(1, QString("Three").toUtf8()); // foo 收到消息
    nc.notify(2, QString("Four").toUtf8());  // bar 收到消息，删除通知 1 的观察者，通知 2 的观察者不受影响

    // [3] 测试把对象序列化和反序列化
    qDebug() << "---------------- 序列化例子 ----------------";
    nc.addObserver(3, &foo);
    Message msg(12345, "你是谁？你从哪里来？你要到哪里去？");
    nc.notify(3, msg.toByteArray());

    // [4] 跨线程发送通知
    qDebug() << "---------------- 跨线程发送通知 ----------------";
    nc.addObserver(4, &foo);
    NotifyThread thread;
    thread.start();

    return app.exec();
}
```

输出:

```
---------------- foo and bar 都能收到通知 1 和 2 ----------------
NotificationID:  1  -  FooObserver:  "One"
NotificationID:  1  -  BarObserver:  "One"
NotificationID:  2  -  BarObserver:  "Two"
---------------- bar 不再收到通知 1 ----------------
NotificationID:  1  -  FooObserver:  "Three"
NotificationID:  2  -  BarObserver:  "Four"
---------------- 序列化例子 ----------------
NotificationID:  3  -  FooObserver:  "Type: 12345, Content: 你是谁？你从哪里来？你要到哪里去？"
---------------- 跨线程发送通知 ----------------
NotifyThread:    NotifyThread(0x7fff519ed910)
NotificationID:  4  -  FooObserver:  "870" - in thread:  QThread(0x7fbfc0c04690)
NotificationID:  4  -  FooObserver:  "345" - in thread:  QThread(0x7fbfc0c04690)
```

## FooObserver.h and FooObserver.cpp
```cpp
#ifndef FOOOBSERVER_H
#define FOOOBSERVER_H

#include <QObject>

class FooObserver : public QObject {
    Q_OBJECT
public:
    explicit FooObserver(QObject *parent = 0);
    Q_INVOKABLE void notified(int notificationId, const QByteArray &data);
};

#endif // FOOOBSERVER_H
```

```cpp
#include "FooObserver.h"
#include <QDebug>
#include <QThread>
#include "Message.h"

FooObserver::FooObserver(QObject *parent) : QObject(parent) {}

void FooObserver::notified(int notificationId, const QByteArray &data) {
    if (3 == notificationId) {
        // notificationId 为 3 说明接收到的通知是 Message
        qDebug() << "NotificationID: " << notificationId << " - " << "FooObserver: " << Message::fromByteArray(data).toString();
    } else if (4 == notificationId) {
        qDebug() << "NotificationID: " << notificationId << " - " << "FooObserver: " << data << "- in thread: " << QThread::currentThread();
    } else {
        qDebug() << "NotificationID: " << notificationId << " - " << "FooObserver: " << data;
    }
}
```

## BarObserver.h and BarObserver.cpp
```cpp
#ifndef BAROBSERVER_H
#define BAROBSERVER_H

#include <QObject>

class BarObserver : public QObject {
    Q_OBJECT
public:
    explicit BarObserver(QObject *parent = 0);
    Q_INVOKABLE void notified(int notificationId, const QByteArray &data);
};

#endif // BAROBSERVER_H
```

```cpp
#include "BarObserver.h"
#include <QDebug>

BarObserver::BarObserver(QObject *parent) : QObject(parent) {}

void BarObserver::notified(int notificationId, const QByteArray &data) {
    qDebug() << "NotificationID: " << notificationId << " - " << "BarObserver: " << data;
}
```

## NotifyThread.h and NotifyThread.cpp
```cpp
#ifndef NOTIFYTHREAD_H
#define NOTIFYTHREAD_H
#include <QThread>

class NotifyThread : public QThread {
    Q_OBJECT
public:
    NotifyThread();

protected:
    void run() Q_DECL_OVERRIDE;
};

#endif // NOTIFYTHREAD_H
```

```cpp
#include "NotifyThread.h"
#include "NotificationCenter.h"
#include <QDebug>
#include <QDateTime>

NotifyThread::NotifyThread() {}

void NotifyThread::run() {
    qDebug() << "NotifyThread: " << QThread::currentThread();
    qsrand(QDateTime::currentDateTime().toMSecsSinceEpoch());

    while (true) {
        NOTIFY_CROSS_THREAD(4, QString::number(qrand() % 900 + 100).toUtf8());

        QThread::msleep(1000);
    }
}
```
