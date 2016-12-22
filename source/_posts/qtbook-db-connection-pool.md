---
title: 数据库连接池
date: 2016-12-22 13:13:16
tags: QtBook
---
在前面的章节里，我们使用了下面的函数创建和取得数据库连接：

```
void createConnectionByName(const QString &connectionName) {
    QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL", connectionName);
    db.setHostName("127.0.0.1");
    db.setDatabaseName("qt"); // 如果是 SQLite 则为数据库文件路径
    db.setUserName("root");   // 如果是 SQLite 不需要
    db.setPassword("root");   // 如果是 SQLite 不需要

    if (!db.open()) {
        qDebug() << "Connect to MySql error: " << db.lastError().text();
        return;
    }
}

QSqlDatabase getConnectionByName(const QString &connectionName) {
    return QSqlDatabase::database(connectionName);
}
```

**虽然抽象出了连接的创建和获取，但是有几个弊端：**

* 需要我们维护连接的名字
* 获取连接的时候需要传入连接的名字
* 获取连接的时候不知道连接是否已经被使用
* 使用多线程的时候，每个线程都必须使用不同的连接，我们必须保证同一个连接不能在多个线程里被同时使用
* 控制创建连接的数量比较困难，因为不能在程序里无限制的创建连接
* 连接断了后不会自动重连
* 删除连接不方便 <!--more-->

为了解决上面的几个问题，这一节我们将实现一个简易的数据库连接池。使用数据库连接池后，连接的创建，获取，释放等只需要使用下面 3 个函数，而且每次获取的连接一定没有被其它线程使用，刚刚提到的那些弊端都通过连接池解决了。

| 功能    | 代码                                       |
| ----- | ---------------------------------------- |
| 获取连接  | QSqlDatabase db = ConnectionPool::openConnection() |
| 释放连接  | ConnectionPool::closeConnection(db)      |
| 关闭连接池 | ConnectionPool::destroy() // 一般在 main() 函数返回前调用 |

## 数据库连接池的使用
在具体介绍数据库连接池的实现之前，先来看看怎么使用。

```cpp
#include "ConnectionPool.h"
#include <QDebug>

void foo() {
    // [1] 从数据库连接池里取得连接
    QSqlDatabase db = ConnectionPool::openConnection();

    // [2] 使用连接查询数据库
    QSqlQuery query(db);
    query.exec("SELECT * FROM user where id=1");

    while (query.next()) {
        qDebug() << query.value("username").toString();
    }

    // [3] 连接使用完后需要释放回数据库连接池
    ConnectionPool::closeConnection(db);
}

int main(int argc, char *argv[]) {
    foo();

    // [4] 释放数据库连接
    ConnectionPool::destroy();
    return 0;
}
```

就像上面程序所示，使用数据库连接池时不需要关系连接的创建，关闭等，只管用。

## 数据库连接池的特点

* 获取连接时不需要了解连接的名字，连接池内部维护连接的名字
* 支持多线程，保证获取到的连接一定是没有被其他线程正在使用
* 按需创建连接
* 可以创建多个连接
* 可以控制连接的数量
* 连接被复用，不是每次都重新创建一个新的连接（连接的创建是一个很消耗资源的过程）
* 连接断开了后会自动重连
* 当无可用连接时，获取连接的线程会等待一定时间尝试继续获取，直到取到有效连接或者超时返回一个无效的连接
* 关闭连接很简单

## 数据库连接池的实现
数据库连接池的实现只需要 2 个文件：`ConnectionPool.h` 和 `ConnectionPool.cpp`，下面列出程序的内容加以介绍。

```cpp
// ConnectionPool.h
#ifndef CONNECTIONPOOL_H
#define CONNECTIONPOOL_H

/**
 * 实现了一个简易的数据库连接池，简化了数据库连接的获取。通过配置最大的连接数可创建多个连接支持多线程访问数据库，
 * Qt 里同一个数据库连接不能被多个线程共享。连接使用完后释放回连接池而不是直接关闭，再次使用的时候不必重新建立连接，
 * 建立连接是很耗时的操作，底层是 Socket 连接。
 *
 * 如果 testOnBorrow 为 true，则连接断开后会自动重新连接（例如数据库程序崩溃了，网络的原因导致连接断了等），
 * 但是每次获取连接的时候都会先访问一下数据库测试连接是否有效，如果无效则重新建立连接。testOnBorrow 为 true 时，
 * 需要提供一条 SQL 语句用于测试查询，例如 MySQL 下可以用 SELECT 1。
 *
 * 如果 testOnBorrow 为 false，则连接断开后不会自动重新连接，这时获取到的连接调用 QSqlDatabase::isOpen() 返回的值
 * 仍然是 true（因为先前的时候已经建立好了连接，Qt 里没有提供判断底层连接断开的方法或者信号）。
 *
 * 当程序结束后，需要调用 ConnectionPool::destroy() 关闭所有数据库的连接（一般在 main() 函数返回前调用）。
 *
 * 使用方法：
 * 1. 从数据库连接池里取得连接
 *    QSqlDatabase db = ConnectionPool::openConnection();
 *
 * 2. 使用 db 访问数据库，如
 *    QSqlQuery query(db);
 *
 * 3. 数据库连接使用完后需要释放回数据库连接池
 *    ConnectionPool::closeConnection(db);
 *
 * 4. 程序结束的时候销毁连接池，关闭所有数据库连接
 *    ConnectionPool::destroy();
 */

class QSqlDatabase;
class ConnectionPoolPrivate;

class ConnectionPool {
public:
    static void destroy(); // 关闭所有的数据库连接
    static QSqlDatabase openConnection();                        // 获取数据库连接
    static void closeConnection(const QSqlDatabase &connection); // 释放数据库连接回连接池

    ~ConnectionPool();

private:
    static ConnectionPool& getInstance();

    ConnectionPool();
    ConnectionPool(const ConnectionPool &other);
    ConnectionPool& operator=(const ConnectionPool &other);

    ConnectionPoolPrivate* d;
    static ConnectionPool *instance;
};

#endif // CONNECTIONPOOL_H
```

* `openConnection()` 用于从连接池里获取连接。
* `closeConnection(const QSqlDatabase &connection)` 并不会真正的关闭连接，而是把连接放回连接池为了复用。连接的底层是通过 Socket 来通讯的，建立 Socket 连接是非常耗时的，如果每个连接都在使用完后就给断开 Socket 连接，需要的时候再重新建立 Socket 连接是非常浪费的，所以要尽量的复用以提高效率。
* `destroy()` 销毁连接池，真正的关闭所有的数据库连接，一般在程序结束的时候才调用，在 main() 函数的 return 语句前。
* `ConnectionPool` 使用了 Singleton 模式，保证在程序运行的时候只有一个对象被创建，`getInstance()` 用于取得这个唯一的对象。一般情况下使用 openConnection() 的方法在 Singleton 模式下的调用应该像这样 ConnectionPool::getInstance().openConnection()，但是我们实现的却是 `ConnectionPool::openConnection()`，因为我们把 `openConnection()` 也定义成静态方法，在它里面调用 `getInstance()` 访问这个对象的数据，这样做的好处即使用了 Singleton 的优势，也简化了 `openConnection()` 的调用。

```cpp
// ConnectionPool.cpp
#include "ConnectionPool.h"
#include <QDebug>
#include <QtSql>
#include <QStack>
#include <QString>
#include <QMutex>
#include <QSemaphore>

/*-----------------------------------------------------------------------------|
 |                          ConnectionPoolPrivate 的定义                        |
 |----------------------------------------------------------------------------*/
class ConnectionPoolPrivate {
public:
    ConnectionPoolPrivate();
    ~ConnectionPoolPrivate();

    QSqlDatabase createConnection(const QString &connectionName); // 创建数据库连接

    QStack<QString> usedConnectionNames;   // 已使用的数据库连接名
    QStack<QString> unusedConnectionNames; // 未使用的数据库连接名

    // 数据库信息
    QString hostName;
    QString databaseName;
    QString username;
    QString password;
    QString databaseType;
    int     port;

    bool    testOnBorrow;    // 取得连接的时候是否验证连接有效
    QString testOnBorrowSql; // 测试访问数据库的 SQL
    int maxWaitTime;         // 获取连接最大等待时间
    int maxConnectionCount;  // 最大连接数

    QSemaphore *semaphore;

    static QMutex mutex;
    static int lastKey; // 用来创建连接的名字，保证连接名字不会重复
};

QMutex ConnectionPoolPrivate::mutex;
int ConnectionPoolPrivate::lastKey = 0;

ConnectionPoolPrivate::ConnectionPoolPrivate() {
    // 创建数据库连接的这些信息在实际开发的时都需要通过读取配置文件得到，
    // 这里为了演示方便所以写死在了代码里。
    hostName     = "127.0.0.1";
    databaseName = "qt";
    username     = "root";
    password     = "root";
    databaseType = "QMYSQL";
    port         = 3306;
    testOnBorrow = true;
    testOnBorrowSql = "SELECT 1";
    maxWaitTime     = 5000;
    maxConnectionCount = 5;

    semaphore = new QSemaphore(maxConnectionCount);
}

ConnectionPoolPrivate::~ConnectionPoolPrivate() {
    // 销毁连接池的时候删除所有的连接
    foreach(QString connectionName, usedConnectionNames) {
        createConnection(connectionName).close();
        QSqlDatabase::removeDatabase(connectionName);
    }

    foreach(QString connectionName, unusedConnectionNames) {
        QSqlDatabase::removeDatabase(connectionName);
    }

    delete semaphore;
}

QSqlDatabase ConnectionPoolPrivate::createConnection(const QString &connectionName) {
    // 连接已经创建过了，复用它，而不是重新创建
    if (QSqlDatabase::contains(connectionName)) {
        QSqlDatabase existingDb = QSqlDatabase::database(connectionName);

        if (testOnBorrow) {
            // 返回连接前访问数据库，如果连接断开，重新建立连接
            qDebug() << QString("Test connection on borrow, execute: %1, for connection %2")
                        .arg(testOnBorrowSql).arg(connectionName);
            QSqlQuery query(testOnBorrowSql, existingDb);

            // 仍然连不上数据库，例如网络问题，数据库关闭
            if (query.lastError().type() != QSqlError::NoError && !existingDb.open()) {
                qDebug() << "Open datatabase error:" << existingDb.lastError().text();
                return QSqlDatabase();
            }
        }

        return existingDb;
    }

    // 创建一个新的数据库连接
    QSqlDatabase db = QSqlDatabase::addDatabase(databaseType, connectionName);
    db.setHostName(hostName);
    db.setDatabaseName(databaseName);
    db.setUserName(username);
    db.setPassword(password);

    if (port != 0) {
        db.setPort(port);
    }

    if (!db.open()) {
        qDebug() << "Open datatabase error:" << db.lastError().text();
        return QSqlDatabase();
    }

    return db;
}

/*-----------------------------------------------------------------------------|
 |                             ConnectionPool 的定义                            |
 |----------------------------------------------------------------------------*/
ConnectionPool* ConnectionPool::instance = NULL;

ConnectionPool::ConnectionPool() {
    d = new ConnectionPoolPrivate();
}

ConnectionPool::~ConnectionPool() {
    delete d;
}

ConnectionPool& ConnectionPool::getInstance() {
    if (NULL == instance) {
        QMutexLocker locker(&ConnectionPoolPrivate::mutex);

        if (NULL == instance) {
            instance = new ConnectionPool();
        }
    }

    return *instance;
}

void ConnectionPool::destroy() {
    QMutexLocker locker(&ConnectionPoolPrivate::mutex);
    delete instance;
    instance = NULL;
}

QSqlDatabase ConnectionPool::openConnection() {
    ConnectionPool& pool = ConnectionPool::getInstance();

    if (pool.d->semaphore->tryAcquire(1, pool.d->maxWaitTime)) {
        // 有已经回收的连接，复用它们
        // 没有已经回收的连接，则创建新的连接
        ConnectionPoolPrivate::mutex.lock();
        QString connectionName = pool.d->unusedConnectionNames.size() > 0 ?
                    pool.d->unusedConnectionNames.pop() :
                    QString("C%1").arg(++ConnectionPoolPrivate::lastKey);
        pool.d->usedConnectionNames.push(connectionName);
        ConnectionPoolPrivate::mutex.unlock();

        // 创建连接，因为创建连接很耗时，所以不放在 lock 的范围内，提高并发效率
        QSqlDatabase db = pool.d->createConnection(connectionName);

        if (!db.isOpen()) {
            ConnectionPoolPrivate::mutex.lock();
            pool.d->usedConnectionNames.removeOne(connectionName); // 删除无效连接
            ConnectionPoolPrivate::mutex.unlock();

            pool.d->semaphore->release(); // 没有消耗连接
        }

        return db;
    } else {
        // 创建连接超时，返回一个无效连接
        qDebug() << "Time out to create connection.";
        return QSqlDatabase();
    }
}

void ConnectionPool::closeConnection(const QSqlDatabase &connection) {
    ConnectionPool& pool = ConnectionPool::getInstance();
    QString connectionName = connection.connectionName();

    // 如果是我们创建的连接，并且已经被使用，则从 used 里删除，放入 unused 里
    if (pool.d->usedConnectionNames.contains(connectionName)) {
        QMutexLocker locker(&ConnectionPoolPrivate::mutex);
        pool.d->usedConnectionNames.removeOne(connectionName);
        pool.d->unusedConnectionNames.push(connectionName);
        pool.d->semaphore->release();
    }
}
```

* `usedConnectionNames` 保存正在被使用的连接的名字，用于保证同一个连接不会同时被多个线程使用。
* `unusedConnectionNames` 保存没有被使用的连接的名字，它们对应的连接在调用 `openConnection()` 时返回。
* 如果 `testOnBorrow` 为 true，则连接断开后会自动重新连接（导致连接断开的情况，例如数据库程序崩溃了，网络断开等都会导致数据库连接断开）。每次获取连接的时候都会先查询一下数据库，如果发现连接无效则重新建立连接。`testOnBorrow` 为 true 时，需要提供一条 SQL 语句用于测试查询，例如 MySQL 下可以用 `SELECT 1`。如果 `testOnBorrow` 为 false，则连接断开后不会自动重新连接。需要注意的是，Qt 里已经建立好的数据库连接当连接断开后调用 QSqlDatabase::isOpen() 返回的值仍然是 true，Qt 里没有提供判断底层连接断开的方法或者信号，所以 QSqlDatabase::isOpen() 返回的仍然是先前的状态 true。
* `testOnBorrowSql` 为测试访问数据库的 SQL，一般是一个非常轻量级的 SQL，如 `SELECT 1`。
* 获取连接的时候，如果没有可用连接，我们的策略并不是直接返回一个无效的连接，而是等待至多 `maxWaitTime` 毫秒，如果期间有连接被释放回连接池里就返回这个连接，没有就继续等待直到 `maxWaitTime` 毫秒仍然没有可用连接才返回一个无效的连接。
* 因为我们不能在程序里无限制的创建连接，用 `maxConnectionCount` 来控制创建连接的最大数量。
* 为了支持多线程，使用了 QMutex，QSemaphore 来保护共享资源 usedConnectionNames 和 unusedConnectionNames 的读写。

在 `ConnectionPoolPrivate` 的构造函数里写死了访问数据库和连接池的配置，为了方便所以都硬编码写在了代码里，实际开发的时候这么做是不可取的，都应该从配置文件里读取，这样当它们变化后只需要修改配置文件就能生效，否则就需要修改代码，然后编译，重新发布等，在后面的 `DBUtil` 章节里就是从配置文件读取的。

**openConnection() 函数相对比较复杂，也是 ConnectionPool 的核心**

1. `pool.d->semaphore->tryAcquire(1, pool.maxWaitTime)` 等待可创建或者有可复用的连接，如果超时仍然没有可用连接，则返回一个无效的连接 QSqlDatabase()。
2. 如果没有可复用连接，但是已经创建的连接数没有达到最大，那么就创建一个新的连接，并把这个连接的名字添加到 `usedConnectionNames`。
3. 如果有可复用的连接，则复用它，把它的名字从 `unusedConnectionNames` 里删除并且添加到 `usedConnectionNames`。



 **createConnection() 是真正创建连接的函数**

1. 如果连接已经被创建，不需要重新创建，而是复用它。`testOnBorrow` 为 true 的话，返回这个连接前会先用 SQL 语句 `testOnBorrowSql` 访问一下数据库，没问题就返回这个连接，如果出错则说明连接已经断开了，需要重新和数据库建立连接。
2. 如果连接没有被创建过，才会真正地建立一个新的连接。



 **closeConnection() 并不是真的断开连接**

1. 需要判断连接是否我们创建的，如果不是就不处理。
2. 把连接的名字从 `usedConnectionNames` 里删除并放到 `unusedConnectionNames` 里，表示这个连接已经被回收，可以被复用了。

## 多线程测试连接池

`测试用例`：连接池允许最多创建 5 个连接，启动 10 个线程用连接池里获取连接访问数据库。

```cpp
// TestConnectionPoolWithMultiThread.h
#ifndef TESTCONNECTIONPOOLWITHMULTITHREAD_H
#define TESTCONNECTIONPOOLWITHMULTITHREAD_H

#include <QThread>

class TestConnectionPoolWithMultiThread : public QThread {
public:
    TestConnectionPoolWithMultiThread(int sn);

protected:
    void run();
    int sn;
};

#endif // TESTCONNECTIONPOOLWITHMULTITHREAD_H
```

```cpp
// TestConnectionPoolWithMultiThread.cpp
#include "TestConnectionPoolWithMultiThread.h"
#include "ConnectionPool.h"
#include <QtSql>

TestConnectionPoolWithMultiThread::TestConnectionPoolWithMultiThread(int sn)  : sn(sn) { }

void TestConnectionPoolWithMultiThread::run() {
    // 从数据库连接池里取得连接
    QSqlDatabase db = ConnectionPool::openConnection();
    qDebug() << QString("In thread %1 run() with connection: %2").arg(sn).arg(db.connectionName());

    QSqlQuery query(db);
    query.exec("SELECT * FROM user where id=1");

    while (query.next()) {
        qDebug() << query.value("username").toString();
    }

    // 连接使用完后需要释放回数据库连接池
    ConnectionPool::closeConnection(db);
}
```

```cpp
// main.cpp
#include "ConnectionPool.h"
#include "TestConnectionPoolWithMultiThread.h"
#include <QApplication>
#include <QPushButton>
#include <QDebug>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    QPushButton *button = new QPushButton("Access Database");
    button->show();

    QObject::connect(button, &QPushButton::clicked, []() {
        for (int i = 0; i < 10; ++i) {
            TestConnectionPoolWithMultiThread *thread = new TestConnectionPoolWithMultiThread(i);
            thread->start();
            // QThread::msleep(50);
        }
    });

    int ret = a.exec();
    ConnectionPool::destroy(); // 程序结束时销毁数据库连接池
    return ret;
}
```

执行程序，点击按钮 `Access Database`，输出如下：
> "In thread 4 run() with connection: C5"  
> "Alice"  
> "Test connection on borrow, execute: SELECT 1, for connection C5"  
> "In thread 5 run() with connection: C5"  
> "Alice"  
> "Test connection on borrow, execute: SELECT 1, for connection C5"  
> "In thread 6 run() with connection: C5"  
> "Alice"  
> "Test connection on borrow, execute: SELECT 1, for connection C5"  
> "In thread 7 run() with connection: C5"  
> "Alice"  
> "Test connection on borrow, execute: SELECT 1, for connection C5"  
> "In thread 9 run() with connection: C5"  
> "Alice"  
> "Test connection on borrow, execute: SELECT 1, for connection C5"  
> "In thread 8 run() with connection: C5"  
> "Alice"  
> "In thread 1 run() with connection: C2"  
> "Alice"  
> "In thread 3 run() with connection: C4"  
> "Alice"  
> "In thread 2 run() with connection: C3"  
> "Alice"  
> "In thread 0 run() with connection: C1"  
> "Alice"

可以看到，线程 0, 1, 2, 3, 4 的连接是新创建的，后面 5 个线程的连接复用了前面创建的连接。线程 5，6，7，8，9 复用了连接 C5，由于我们采用的策略是复用归还时间短的连接，这样越早归还的连接的不活动时间就可能越长，以后就可以实现当连接的不活动时间达到一定的时候就从连接池里删除，减少资源的浪费。可以再做一下几个测试，看看连接池是否都能正确的运行与观察连接的复用情况。

**Case 1**

1. 点击按钮 `Access Database`，正常输出。
2. 然后关闭数据库，点击按钮 `Access Database`，应该提示连不上数据库。
3. 启动数据库，点击按钮 `Access Database`，正常输出。

**Case 2**

* 把线程数增加到 100 个，1000 个。
* 同时测试关闭和再次打开数据库。
* 观察连接的复用情况

**Case 3**

* 在线程的 run() 函数里随机等待一段时间，例如 0 到 100 毫秒。
* 观察连接的复用情况

## 思考

数据库连接池的功能基本已经完成，但还是不完善。考虑一下如果我们设置最大连接数为 100，高峰期访问比较多，创建满了 100 个连接，但是当闲置下来后例如晚上 3 点可能只需要 2 个连接，其余 98 个连接都长时间不用，但它们一直都和数据库保持着连接，这对资源（Socket 连接）是很大的浪费。需要有这样的机制，当发现连接一段时间没有被使用后就把其关闭，并从 `unusedConnectionNames` 里删除。还有例如连接被分配后没有释放回连接池，即一直在 `usedConnectionNames` 里面，造成连接泄漏，所以有必要超过一定时间后连接池应该主动把其回收。怎么实现这些的功能，这里就不在一一说明，大家独自思考一下应该怎么实现吧。