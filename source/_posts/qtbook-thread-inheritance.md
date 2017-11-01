---
title: 继承 QThread 实现多线程
date: 2017-10-29 12:09:15
tags: QtBook
---

Qt 中使用多线程，最简单直观的方法就是继承 `QThread`，重写 `run()` 方法，需要使用多线程执行的代码放在 `run()` 函数中，调用 `start()` 函数启动线程，线程正在运行时 `isRunning()` 返回 true，结束运行后发出信号 `finished()`。

##  实现线程

仍以读取文本显示到 QTextEdit 为例，类 ReadingThread 继承 QThread，在 run() 方法中读取文件并添加到 QTextEdit。

```cpp
// 文件名: ReadingThread.h

#ifndef READINGTHREAD_H
#define READINGTHREAD_H

#include <QThread>

class QTextEdit;

class ReadingThread : public QThread {
public:
    ReadingThread(QTextEdit *textEdit, QObject *parent = NULL);

protected:
    void run() Q_DECL_OVERRIDE;

private:
    QTextEdit *textEdit;
};

#endif // READINGTHREAD_H
```

```cpp
// 文件名: ReadingThread.cpp

#include "ReadingThread.h"
#include <QFile>
#include <QTextStream>
#include <QTextEdit>
#include <QMetaObject>

ReadingThread::ReadingThread(QTextEdit *textEdit, QObject *parent) : QThread(parent), textEdit(textEdit) {

}

void ReadingThread::run() {
    QFile file("/Users/Biao/Desktop/data.txt");

    if (!file.open(QIODevice::Text | QIODevice::ReadOnly)) {
        return;
    }

    QTextStream in(&file);
    while (!in.atEnd()) {
        QString line = in.readLine();
        textEdit->append(line);
    }
}
```

<!--more-->在点击按钮的槽函数中启动线程:

```cpp
ReadingWidget::ReadingWidget(QWidget *parent) : QWidget(parent), ui(new Ui::ReadingWidget) {
    ui->setupUi(this);

    // 创建线程对象
    readingThread = new ReadingThread(ui->textEdit, this);

    // 点击 "开始读取" 按钮启动线程
    connect(ui->startButton, &QPushButton::clicked, [this] {
        if (!readingThread->isRunning()) {
            // 线程没有运行时才启动，调用 start() 启动线程
            readingThread->start();
        } else {
            qDebug() << "线程正在运行";
        }
    });

    // 线程结束时的信号槽
    connect(readingThread, &QThread::finished, [] {
        qDebug() << "线程结束运行";
    });
}
```

运行程序，点击 `开始读取` 按钮，文本显示到了 text edit 中，但是控制台很可能输出一句提示

> QObject::connect: Cannot queue arguments of type 'QTextCursor'
> (Make sure 'QTextCursor' is registered using qRegisterMetaType().)

也许你觉得这没什么，但是当读取大文件时，如 **qtgui.index**，程序就直接崩溃了，到底发生了什么，相信到这里，90% 的人都不知道为啥，这是一个非常隐蔽的问题：Qt 中一个线程里不能直接调用另一个线程的对象的函数，解决这个问题很简单，把 `textEdit->append(line)` 替换为 `QMetaObject::invokeMethod(textEdit, "append", Q_ARG(QString, line))` 即可，想要知道为什么，请参考 [线程一调用线程二中函数的正确姿势](/qtbook-thread-call-in-different-thread)。

程序不会崩溃了，此外又发现了一个问题，界面上有时候只显示了文件的部分内容，然后就冻住了，过了几分钟才全部显示，不是说好的耗时任务在线程中执行就是为了不阻塞 UI 线程么，怎么还是阻塞了？

如果观察控制台，可以看到 1s 左右就输出了 **线程结束运行**，说明在新线程中读取文件是正确的，但是每读取一行就会让 UI 线程执行一个更新界面的操作，qtgui.index 有 18.5 万行，导致 UI 线程瞬间累积了 18.5 万个更新界面的操作，要知道更新界面是非常消耗资源的，所以才导致 UI 忙不过来被冻住。为了减少 UI 线程更新界面的频率，读取一行后暂停 1ms，就能看到内容不停的被添加到 text edit 中，证明了读取文件的线程没有阻塞 UI 线程，代码如下：

```cpp
void ReadingThread::run() {
    QFile file("/Users/Biao/Qt5.9.2/Docs/Qt-5.9.2/qtgui/qtgui.index");

    if (!file.open(QIODevice::Text | QIODevice::ReadOnly)) {
        return;
    }

    QTextStream in(&file);
    while (!in.atEnd()) {
        QString line = in.readLine();
        QMetaObject::invokeMethod(textEdit, "append", Q_ARG(QString, line));
        QThread::msleep(1);
    }
}
```

## 结束线程

上面读取文件的过程太过漫长，想随时结束读取线程，相信大多数人会调用 `terminate()` 函数来结束线程：

```cpp
// 点击 "结束读取" 按钮结束线程
connect(ui->stopButton, &QPushButton::clicked, [this] {
    readingThread->terminate();
    readingThread->wait();
});
```

> `void QThread::terminate()`
>
> Terminates the execution of the thread. The thread may or may not be terminated immediately, depending on the operating system's scheduling policies. Use QThread::wait() after terminate(), to be sure.
>
> **Warning**: This function is dangerous and its use is discouraged. The thread can be terminated at any point in its code path. Threads can be terminated while modifying data. There is no chance for the thread to clean up after itself, unlock any held mutexes, etc. In short, use this function only if absolutely necessary.

需要特别注意，使用 `terminate()` 结束线程是非常危险的，它可能在 `run()` 中的任意地方结束，可能导致死锁、资源没有释放等，虽然在我们这个例子中使用 `terminate()` 结束线程没问题，为了养成好的习惯，还是不要随意使用的好。

结束线程其实非常简单，只要 `run()` 函数执行结束即可。**推荐结束线程的方法是：定义一个变量，用它来控制 `run()` 中循环的结束。**下面定义了 bool 变量 stopped，在线程执行的时候，当其为 true 时循环就会结束，然后 run() 函数结束返回，线程就结束运行了。

```cpp
// 文件名: ReadingThread.h

#ifndef READINGTHREAD_H
#define READINGTHREAD_H

#include <QThread>

class QTextEdit;

class ReadingThread : public QThread {
public:
    ReadingThread(QTextEdit *textEdit, QObject *parent = NULL);
    void stop(); // 结束线程

protected:
    void run() Q_DECL_OVERRIDE;

private:
    bool stopped; // 线程结束的标志
    QTextEdit *textEdit;
};

#endif // READINGTHREAD_H
```

```cpp
// 文件名: ReadingThread.cpp

#include "ReadingThread.h"
#include <QFile>
#include <QTextStream>
#include <QTextEdit>
#include <QMetaObject>

ReadingThread::ReadingThread(QTextEdit *textEdit, QObject *parent) : QThread(parent), textEdit(textEdit) {

}

void ReadingThread::stop() {
    stopped = true;
}

void ReadingThread::run() {
    stopped = false; // 线程开始执行时设置 stopped 为 false
    QFile file("/Users/Biao/Qt5.9.2/Docs/Qt-5.9.2/qtgui/qtgui.index");

    if (!file.open(QIODevice::Text | QIODevice::ReadOnly)) {
        return;
    }

    QTextStream in(&file);

    // 当 stopped 为 true，或者 atEnd() 为 true 时结束 while 循环
    while (!stopped && !in.atEnd()) {
        QString line = in.readLine();
        QMetaObject::invokeMethod(textEdit, "append", Q_ARG(QString, line));
        QThread::msleep(1);
    }
}
```

调用 `stop()` 函数结束线程时，最好再调用一下 `wait()` 等待线程真的结束，因为调用 `stop()` 后需要等一下 ReadingThread 才能得到执行权限，然后循环结束，等到 `run()` 结束返回时需要一点时间，虽然非常非常短暂，但对于计算机来说足够发生很多事了，尤其是高并发的情况下更是不可预料，一定要重视小概率事件，否则很多时候死得莫名其妙的，为了等待线程彻底结束，调用 `wait()` 还是很有必要的：

```cpp
// 点击 "结束读取" 按钮结束线程
connect(ui->stopButton, &QPushButton::clicked, [this] {
    readingThread->stop(); // 提示结束线程执行
    readingThread->wait(); // 等待线程真正的结束执行
});
```

## 重用线程

线程结束运行后仍然可以再次调用 `start()` 重新启动。使用这个特性，可以重复利用线程执行任务，而不是每次都创建一个新的线程，创建线程需要和操作系统打交道，是很消耗资源的操作，所以能够重用就尽量重用，Qt 提供了线程池 QThreadPool 就是为了重复利用线程，避免大量的创建线程，提高程序的效率。

## 总结

本节使用简单的例子介绍了继承 QThread 实现多线程，在新线程中执行耗时任务，强调不要随意使用 `terminate()` 结束线程并给出了结束线程的简单方法，最后提示为了提高程序效率，应尽量的重复利用线程，避免大量创建新线程浪费资源。