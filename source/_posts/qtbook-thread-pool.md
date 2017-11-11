---
title: 线程池 QThreadPool
date: 2017-11-11 20:07:36
tags: QtBook
---

创建线程需要向系统申请资源，线程切换时操作系统会切换线程上下文，可能会从用户态切换到内核态，当有很多线程时，频繁地切换线程会导致消耗大量的 CPU 以及内核资源，真正用于计算的资源就减少了，反而会降低程序的效率。线程并不是越多越好，线程池的作用是管理、复用、回收一组线程，控制线程的数量，避免频繁的创建和销毁线程而浪费资源。

Qt 中的线程池类为 QThreadPool，每一个 Qt 程序都有一个全局的线程池，调用 `QThreadPool::globalInstance()` 得到，它默认最多创建 8 个线程，如果想改变最大线程数则调用 `setMaxThreadCount()` 进行修改，调用 `activeThreadCount()` 查看线程池中当前活跃的线程数。

使用线程池挺简单的，定一个任务类例如叫 Task，继承 QRunnable 并实现虚函数 `run()`，Task 的对象作为 `QThreadPool::start()` 的参数就可以了，线程池会自动的在线程中调用 Task 的 run() 函数，异步执行。线程池中的 QRunnable 对象太多时并不会为立即为每一个 QRunnable 对象创建一个线程，而是让它们排队执行，同时最多有 `maxThreadCount()` 个线程并行执行。

提交给线程池的 QRunnable 对象在它的 run() 函数执行完后会被自动 delete 掉，如果不想线程池删除它，在调用线程池的 start() 前调用 `setAutoDelete(false)` 即可。

<!--more-->

为了演示线程池的使用，下面定义一个任务类 Task，属性 id 为了便于了解任务对象的创建、执行、销毁，run() 模拟耗时操作，随机执行 [500, 2500] 毫秒，然后在 main() 函数中创建 100 个 Task 对象提交给线程池，从输出中可以看到同时只有 8 个任务在执行，run() 执行结束后任务对象被 delete 掉了。

```cpp
// 文件名: Task.h
#ifndef TASK_H
#define TASK_H

#include <QRunnable>

class Task : public QRunnable {
public:
    Task(int id);
    ~Task();

    void run() Q_DECL_OVERRIDE;

private:
    int id; // 线程的 ID
};

#endif // TASK_H
```

```cpp
// 文件名: Task.cpp
#include "Task.h"
#include <QDebug>
#include <QThread>
#include <QDateTime>

Task::Task(int id) : id(id) {

}

Task::~Task() {
    qDebug().noquote() << QString("~Task() with ID %1").arg(id); // 方便查看对象是否被 delete
}

void Task::run() {
    qDebug().noquote() << QString("Start thread %1 at %2").arg(id).arg(QDateTime::currentDateTime().toString("mm:ss.z"));
    QThread::msleep(500 + qrand() % 2000); // 每个 run() 函数随机执行 [55, 2500] 毫秒，模拟耗时任务
    qDebug().noquote() << QString("End   thread %1 at %2").arg(id).arg(QDateTime::currentDateTime().toString("mm:ss.z"));
}
```

```cpp
// 文件名: main.cpp
#include <QApplication>
#include <QThreadPool>
#include "Task.h"

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    for (int i = 1; i <= 100; ++i) {
        Task *task = new Task(i); // 创建任务
        QThreadPool::globalInstance()->start(task); // 提交任务给线程池，在线程池中执行
    }

    return a.exec();
}

```

控制台输出如下:

```
Start thread 1 at 27:05.541
Start thread 6 at 27:05.541
Start thread 5 at 27:05.541
Start thread 2 at 27:05.541
Start thread 3 at 27:05.541
Start thread 8 at 27:05.541
Start thread 7 at 27:05.541
Start thread 4 at 27:05.541
End   thread 6 at 27:06.854
~Task() with ID 6
End   thread 1 at 27:06.854
~Task() with ID 1
End   thread 8 at 27:06.854
~Task() with ID 8
End   thread 3 at 27:06.854
~Task() with ID 3
End   thread 7 at 27:06.854
End   thread 4 at 27:06.854
End   thread 2 at 27:06.854
~Task() with ID 7
End   thread 5 at 27:06.854
~Task() with ID 4
~Task() with ID 2
~Task() with ID 5
Start thread 9 at 27:06.855
Start thread 10 at 27:06.855
Start thread 11 at 27:06.855
Start thread 12 at 27:06.855
Start thread 13 at 27:06.855
```

**继承 QThread 和使用 QThreadPool 都能进行多线程编程，那么什么时候使用线程池，什么时候继承 QThread 创建线程呢？**

频繁创建、耗时短的任务使用线程池来执行更合适，例如通过串口不停地接收到数据，然后交给线程池进行计算处理，如果每接收到一次数据都创建一个新线程就太浪费资源了。耗时长的任务就一般会继承 QThread 使用多线程，例如使用 QProcess 启动一个命令行，和命令行进行交互时就可以这么做。

