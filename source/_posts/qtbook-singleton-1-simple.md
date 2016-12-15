---
title: 单例的简单实现
date: 2016-12-15 18:25:19
tags: QtBook
---
用简单直观的方式来实现一个单例的类 ConfigUtil，这里不使用宏，模版等技术，先了解实现一个单例类的理论知识，然后在此基础之上进行思考，优化，最终让我们的实现真正的达到实用的目的，而不只是功能上可用，但是质量却很不好。

##### 实现单例时，需要注意以下几点：
* C++ 的书里经常强调：一个类，至少要提供`构造函数`，`拷贝构造函数`，`析构函数`，`赋值运算操作符`，尤其是有成员变量是指针类型，保存指针的数组或集合时更是需要注意（实现深拷贝）
* 要限制创建和删除 ConfigUtil 的对象
  * 构造函数定义为 private 的，是为了防止其他地方使用 new 创建 ConfigUtil 的对象
  * 析构函数定义为 private 的，是为了防止其他地方使用 delete 删除 ConfigUtil 的对象
  * 拷贝构造函数定义为 private 的，是为了防止通过拷贝构造函数创建新的 ConfigUtil 对象
  * 赋值运算操作符定义为 private 的，是为了防止通过赋值操作创建新的 ConfigUtil 对象
* 通过 `ConfigUtil::getInstance()` 获取 ConfigUtil 的对象
* 当程序结束的时候调用 `ConfigUtil::release()` 删除它的对象，否则会造成内存泄漏。虽然程序结束了，内存会被系统回收，但是理论上还是要保证谁分配的内存谁回收<!--more-->

## ConfigUtil.h

```cpp
#ifndef CONFIGUTIL_H
#define CONFIGUTIL_H

#include <QMutex>
#include <QString>

class ConfigUtil {
public:
    // 这个函数和单例的实现无关，是 ConfigUtil 的功能函数
    QString getDatabaseName() const; 
    
    /**
     * @brief 获取 ConfigUtil 的唯一对象
     * @return ConfigUtil 的对象
     */
    static ConfigUtil& getInstance();

    /**
     * @brief 删除 ConfigUtil 的唯一对象
     */
    static void release();

private:
    ConfigUtil();  // 构造函数
    ~ConfigUtil(); // 析构函数
    ConfigUtil(const ConfigUtil &other); // 拷贝构造函数
    ConfigUtil& operator=(const ConfigUtil &other); // 赋值运算操作符

    static QMutex mutex;
    static ConfigUtil *instance; // ConfigUtil 全局唯一的变量
};

#endif // CONFIGUTIL_H
```

## ConfigUtil.cpp

```cpp
#include "ConfigUtil.h"
#include <QDebug>

QMutex ConfigUtil::mutex;
ConfigUtil* ConfigUtil::instance = 0;

ConfigUtil::ConfigUtil() {
    qDebug() << "ConfigUtil()";
}

ConfigUtil::~ConfigUtil() {
    qDebug() << "~ConfigUtil()";
}

ConfigUtil& ConfigUtil::getInstance() {
    if (0 == instance) {
        mutex.lock();
        if (0 == instance) {
            instance = new ConfigUtil();
        }
        mutex.unlock();
    }

    return *instance;
}

void ConfigUtil::release() {
    if (0 != instance) {
        mutex.lock();
        delete instance;
        instance = 0;
        mutex.unlock();
    }
}

QString ConfigUtil::getDatabaseName() const {
    return "Avatar";
}
```

为了支持多线程，在 getInstance() 和 release() 里使用 mutex 防止线程同步问题。getInstance() 中用了 2 个 if (0 == instance) 是为了保证只创建 1 个 ConfigUtil 的对象，同时保证效率，也可以像下面这样实现，看上去好像舒服了多了：

```cpp
ConfigUtil& ConfigUtil::getInstance() {
    mutex.lock();
    if (0 == instance) {
        instance = new ConfigUtil();
    }
    mutex.unlock();

    return *instance;
}
```

但是，每次调用 ConfigUtil::getInstance() 的时候都要加锁，效率是很低的，具体的细节不是这里的重点，就不作介绍了，在多线程的章节再作进一步讨论。

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

    ConfigUtil::release(); // 程序结束时需要手动析构 ConfigUtil 的对象

    return 0;
}
```

输出：
>ConfigUtil()  
>"Avatar"  
>"Avatar"  
>~ConfigUtil()

从输出可以看到，只有一个 ConfigUtil 的对象被创建，因为构造函数只执行了一次，main() 函数返回前，需要调用 `ConfigUtil::release()` 删除 ConfigUtil 的唯一指针对象 instance。

## 缺点
只有一个单例时还好，夸张一点，如果我们的系统很复杂，需要用到几十，几百个单例的类呢，每一个单例的类都要调用它的 release() 函数来删除对象，是不是不够方便？甚至还有可能会漏掉某些 release() 的调用，这个就不好了。

## 思考
那么，可不可以不需要我们手动的调用 release() 函数，程序结束前自动的删除单例类的对象呢？
