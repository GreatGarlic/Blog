---
title: 线程一中调用线程二中创建的对象的函数的正确姿势
date: 2016-11-08 16:50:49
tags: Qt
---
Qt 里 `线程一的上下文中` 调用 `线程二的上下文中` 的函数的正确姿势是使用 `信号槽` 或者 `QMetaObject::invokeMethod()`，有意思的是 Qt 4 时线程一中直接调用线程二中的函数，程序会直接奔溃退出，很容易发现问题，但在 Qt 5 里有时候没问题，有时候会在控制台有警告，有时候程序会退出，很是莫名其妙，所以最好的办法就是不要直接调用，下面的程序展示了相关测试代码。

> Because of limitations inherited from the low-level libraries on which Qt's GUI support is built, QWidget and its subclasses are not reentrant. One consequence of this is that we cannot directly call functions on a widget from a secondary thread. If we want to, say, change the text of a QLabel from a secondary thread, we can emit a signal connected to QLabel::setText() or call QMetaObject::invokeMethod() from that thread. For example:
>
```
void MyThread::run() {
    ...
    QMetaObject::invokeMethod(label, SLOT(setText(const QString &)), Q_ARG(QString, "Hello"));
    ...
}
```

<!--more-->

## main.cpp
```cpp
#include "Widget.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    Widget w;
    w.show();

    return a.exec();
}
```

## XThread 类
```cpp
#ifndef XTHREAD_H
#define XTHREAD_H

#include <QThread>

class XThread : public QThread {
    Q_OBJECT
public:
    XThread();

protected:
    void run() Q_DECL_OVERRIDE;

signals:
    void currentTime(const QString &time);
};

#endif // XTHREAD_H
```

```cpp
#include "XThread.h"
#include <QDateTime>
#include <QDebug>

XThread::XThread() {
}

void XThread::run() {
    qDebug() << "XThread:  " << QThread::currentThread();

    while (true) {
        emit currentTime(QDateTime::currentDateTime().toString("yyyy-MM-dd HH:mm:ss"));
        QThread::msleep(1000);
    }
}
```

## Widget 类
```cpp
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

class XThread;
class QLabel;

class Widget : public QWidget {
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

    Q_INVOKABLE void showTime(const QString &time);

private:
    XThread *thread;
    QLabel *timeLabel;
};

#endif // WIDGET_H
```

```cpp
#include "Widget.h"
#include "XThread.h"
#include <QDebug>
#include <QMetaObject>
#include <QLabel>
#include <QHBoxLayout>

Widget::Widget(QWidget *parent) : QWidget(parent) {
    // 界面布局
    timeLabel = new QLabel("");
    QHBoxLayout *hl = new QHBoxLayout();
    hl->addWidget(new QLabel("线程里的信号触发修改:"));
    hl->addWidget(timeLabel);
    this->setLayout(hl);

    // 创建启动线程
    thread = new XThread();
    thread->start();

    // 事件处理 1
    connect(thread, &XThread::currentTime, [this](const QString &time) {
        qDebug() << "connect:  " << QThread::currentThread(); // 当前环境的上下文属于线程 XThread
        this->showTime(time); // Error: 有时候没问题，有时候会有警告，有的时候程序直接退出，所以不要这么做，相当于在 XThread 中直接调用
        QMetaObject::invokeMethod(this, "showTime", Q_ARG(QString, time)); // OK: 一个线程调中用另外一个线程中函数的正确姿势
    });

    // 事件处理 2
    connect(thread, &XThread::currentTime, this, &Widget::showTime); // OK: 使用信号槽
}

Widget::~Widget() {
}

void Widget::showTime(const QString &time) {
    qDebug() << "showTime: " << QThread::currentThread(); // 使用 invokeMethod() 调用时属于 Ui 线程
    this->timeLabel->setText(time);
}
```

输出，发现有 2 个线程: 

* 0x7fdcf660e040 是 XThread 
* 0x7fdcf65001e0 是 Ui 线程

```
XThread:   XThread(0x7fdcf660e040)
connect:   XThread(0x7fdcf660e040)
showTime:  XThread(0x7fdcf660e040)
connect:   XThread(0x7fdcf660e040)

showTime:  XThread(0x7fdcf660e040)
showTime:  QThread(0x7fdcf65001e0)
showTime:  QThread(0x7fdcf65001e0)
showTime:  QThread(0x7fdcf65001e0)
showTime:  QThread(0x7fdcf65001e0)
connect:   XThread(0x7fdcf660e040)
showTime:  XThread(0x7fdcf660e040)
showTime:  QThread(0x7fdcf65001e0)
showTime:  QThread(0x7fdcf65001e0)
connect:   XThread(0x7fdcf660e040)
showTime:  XThread(0x7fdcf660e040)
showTime:  QThread(0x7fdcf65001e0)
```

## 结果
```
// 在 XThread 线程上下文里调用
this->showTime(time);

// 在 Ui 线程上下文里调用
QMetaObject::invokeMethod(this, "showTime", Q_ARG(QString, time));
connect(thread, &XThread::currentTime, this, &Widget::showTime);
```

