---
title: 集成自定义类型到 MetaType 系统
date: 2016-11-30 11:33:25
tags: Qt
---

Qt 提供的类型如 QString, QSize 和 QColor 等可以存储在 QVariant 里，并且作为 QObject 及其子类的属性使用，例如调用 `obj.setProperty("name", QString("道格拉斯·狗"))`, `obj.property("name")`，也可以在信号槽中作为参数传递。我们自定义的类如果也想要这么使用，需要使用 Q_DECLARE_METATYPE 和 qRegisterMetaType 来注册后才行。

<!--more-->

## 自定义类型和 QVariant 转换

如果要 QVariant 支持自定义类型，需要使用 Q_DECLARE_METATYPE 注册一下自定义类型

```cpp
#include <QDebug>
#include <QObject>

class Foo {
public:
    int amount;
};

Q_DECLARE_METATYPE(Foo) // [1]

///////////////////////////////////////////////////////////
///                         main()                       //
///////////////////////////////////////////////////////////
int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    Foo foo;
    foo.amount = 123;

    // [2] Foo 的对象和 QVariant 进行转换
    QVariant var = QVariant::fromValue(foo);
    qDebug() << var.value<Foo>().amount;

    return 0;
}
```

> 使用 Q_DECLARE_METATYPE(Foo) 后就能使用 QVariant 存储自定义类型 Foo 了

## 自定义类型作为 QObject 的属性

自定义类型想作为 QObject 的属性使用，除了调用 Q_DECLARE_METATYPE(Foo) 之外，还需要实现运算符 QVariant

```cpp
#include <QDebug>
#include <QObject>

class Foo {
public:
    // [2]
    operator QVariant() const {
        return QVariant::fromValue(*this);
    }

    int amount;
};

Q_DECLARE_METATYPE(Foo) // [1]

///////////////////////////////////////////////////////////
///                         main()                       //
///////////////////////////////////////////////////////////
int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    Foo foo;
    foo.amount = 123;

    // [3] Foo 的对象作为 property 使用
    QObject obj;
    obj.setProperty("foo", foo);
    qDebug() << obj.property("foo").value<Foo>().amount;

    return 0;
}
```

## 信号槽中使用自定义类型

调用 Q_DECLARE_METATYPE(Foo) 之后，Foo 可以在连接类型为 Qt::DirectConnection 的信号槽中使用了，也就是同一个线程中，如果要在多线程的信号槽中使用自定义类型，还需要调用 `qRegisterMetaType<Foo>()`，一般都是在 main() 函数中调用或者在某个构造函数中调用。

```cpp
#include <QDebug>
#include <QObject>

class Foo {
public:
    int amount;
};

Q_DECLARE_METATYPE(Foo) // [1]

///////////////////////////////////////////////////////////
///                         main()                       //
///////////////////////////////////////////////////////////
int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    qRegisterMetaType<Foo>(); // [2]
    connect(...) // [3] 多线程的信号槽，Foo 的对象作为参数

    return 0;
}
```

