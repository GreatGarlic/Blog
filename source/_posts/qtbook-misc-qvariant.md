---
title: 自定义类与 QVariant
date: 2017-03-18 11:18:33
tags: QtBook
---

**QVariant** 非常重要，可以存储很多种不同的类型，例如 **int, QString, QRect, QPoint** 等，其构造函数有很多个，参数是很多种不同的常用类型，还内置了可以直接转换 QVaraint 到某些类型的函数，如 **toInt(), toString(), toPoint(), toSize()** 等，还是 QObject 动态 property 机制的关键，除了支持内置的类型外，QVariant 还被设计成可以存储我们自己定义的类型。

**关键术语:**

* Q_DECLARE_METATYPE
* qRegisterMetaType
* qRegisterMetaTypeStreamOperators
* operator QVariant()


<!--more-->

## 自定义类型和 QVariant 互相转换

想要能够使得自定义类型的对象和 QVariant 能够互相转换，自定义类型需要在类声明的头文件中使用宏 **Q_DECLARE_METATYPE()** 声明一下，告知 Qt 的 Meta System:

* 使用 `QVariant::fromValue(customClassObject)` 把自定义类型的对象转换为 QVariant 对象
* 使用 `variantObject.value<CustomClass>()` 把 QVariant 对象转换为自定义类型的对象

> Adding a Q_DECLARE_METATYPE() makes the type known to all template based functions, including QVariant.

下面使用自定义类 User 和 QVaraint 互转为例

```cpp
// 文件名: User.h
#ifndef USER_H
#define USER_H

#include <QString>
#include <QVariant>
#include <QMetaType>

class User {
public:
    User(int id = 50);

    int id;
    QString username;
    QString password;
};

Q_DECLARE_METATYPE(User) // [1] 向 Qt Meta System 声明

#endif // USER_H
```

```cpp
// 文件名: User.cpp
#include "User.h"

User::User(int id) : id(id) {
}
```

```cpp
// 文件名: main.cpp
#include <QDebug>
#include <QVariant>

#include "User.h"

int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    // [2] 对象转换为 QVariant
    User ali(33);
    QVariant var = QVariant::fromValue(ali);
    // 只有使用 Q_DECLARE_METATYPE 声明过的类 canConvert 才返回 true
    qDebug() << var.canConvert<User>(); // 输出 true

    // [3] QVarint 转换为对象
    User alex = var.value<User>();
    qDebug() << alex.id; // 输出 33

    return 0;
}
```

## 信号槽中使用自定义类型

如果 connect 的类型是 **Qt::DirectConnection**，那么不需要做什么，自定义类型的对象可以直接在信号槽中作为参数，但是如果 connect 的类型是 **Qt::QueuedConnection**，自定义类型除了使用宏 **Q_DECLARE_METATYPE()** 声明外，还必须调用 **qRegisterMetaType** 注册后才可以:

```cpp
qRegisterMetaType<User>(); // 注意: qRegisterMetaType 是函数，不是宏
```

> Adding a Q_DECLARE_METATYPE() makes the type known to all template based functions, including QVariant. Note that if you intend to use the type in queued signal and slot connections or in QObject's property system, you also have to call qRegisterMetaType() since the names are resolved at runtime.
>
> To use the type T in QVariant, using Q_DECLARE_METATYPE() is sufficient. To use the type T in queued signal and slot connections, qRegisterMetaType<T>() must be called before the first connection is established.
>
> Also, to use type T with the QObject::property() API, qRegisterMetaType<T>() must be called before it is used, typically in the constructor of the class that uses T, or in the main() function.

## QObject property 中使用自定义类型

**QObject::setProperty(const char *name, const QVariant &value)** 可以动态地存储数据，这样我们就不需要为了保存数据而定义很多变量了，使用 **QObject::property(const char *name)** 就能够取得存储的数据，是不是非常方便！

为了让 QObject 的 property 支持自定义类型的对象，自定义的类需要实现运算符 **QVariant()**，如下 User 的实现:

```cpp
// 文件名: User.h
#ifndef USER_H
#define USER_H

#include <QString>
#include <QVariant>
#include <QMetaType>

class User {
public:
    User(int id = 50);
    operator QVariant() const; // [1] 为了支持 QObject 的 property 特性

    int id;
    QString username;
    QString password;
};

Q_DECLARE_METATYPE(User)

#endif // USER_H
```

```cpp
// 文件名: User.cpp
#include "User.h"

User::User(int id) : id(id) {
}

User::operator QVariant() const {
    return QVariant::fromValue(*this);
}
```

```cpp
// 文件名: main.cpp
#include <QDebug>
#include <QVariant>

#include "User.h"

int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    User alex(33);
    QObject obj;
    obj.setProperty("user", alex); // [2] 存储自定义类型的对象
    User ronnie = obj.property("user").value<User>(); // [3] 取得自定义类型的对象
    qDebug() << ronnie.id; // 输出 33

    return 0;
}
```

## QDataStream 序列化自定义类型的 QVariant

自定义类型的对象转换为 QVaraint 对象后，如果要使用 QDataStream 操作这个 varaint 的话，需要:

* 自定义类型实现 QDataStream& operator<<(QDataStream &stream, const CustomClass &obj)
* 自定义类型实现 QDataStream& operator>>(QDataStream &stream, CustomClass &obj)
* 使用 qRegisterMetaTypeStreamOperators 进行注册

任然以 User 为例:

```cpp
// 文件名: User.h
#ifndef USER_H
#define USER_H

#include <QString>
#include <QVariant>
#include <QMetaType>
#include <QDataStream>

class User {
public:
    User(int id = 50, const QString &username = QString(), const QString &password = QString());

    friend QDataStream& operator<<(QDataStream &stream, const User &user);
    friend QDataStream& operator>>(QDataStream &stream, User &user);

    int id;
    QString username;
    QString password;
};

Q_DECLARE_METATYPE(User)

#endif // USER_H
```

```cpp
// 文件名: User.cpp
#include "User.h"

User::User(int id, const QString &username, const QString &password)
    : id(id), username(username), password(password) {
}

QDataStream& operator<<(QDataStream &stream, const User &user) {
    stream << user.id << user.username << user.password;
    return stream;
}

QDataStream& operator>>(QDataStream &stream, User &user) {
    stream >> user.id >> user.username >> user.password;
    return stream;
}
```

```cpp
// 文件名: main.cpp
#include <QDebug>
#include <QVariant>
#include <QByteArray>
#include <QDataStream>

#include "User.h"

int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    // [1] QDataStream 操作自定义类型的 QVariant 时需要先注册
    qRegisterMetaTypeStreamOperators<User>();

    // [2] 把对象转换为 QVariant
    QVariant aliVar = QVariant::fromValue(User(100, "Ali", "Secret"));
    QByteArray buffer; // 存储的载体

    // [3] 把 User 的 QVariant 写入 buffer
    QDataStream out(&buffer, QIODevice::WriteOnly);
    out << aliVar;

    // [4] 从 buffer 中读取对象
    QDataStream in(&buffer, QIODevice::ReadOnly);
    QVariant readedAliVar;
    in >> readedAliVar;
    User readedAli = readedAliVar.value<User>();

    // 输出: "ID: 100, Username: Ali, Password: Secret"
    qDebug() << QString("ID: %1, Username: %2, Password: %3")
                .arg(readedAli.id).arg(readedAli.username).arg(readedAli.password);

    return 0;
}
```

