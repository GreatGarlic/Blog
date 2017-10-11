---
title: Lambda 在 Qt 中的运用
date: 2016-09-04 20:05:05
tags: Qt
---

传统的信号槽绑定时，需要先声明槽函数，然后实现槽函数(槽函数的声明和实现需要分别在 .h 和 .cpp 文件中)，最后使用 connect() 绑定起来，而且在 connect() 的时候如果槽函数写错了编译时不会报错，只有在 Debug 模式下运行时才会提示槽函数不存在，Release 模式下运行时不会给予任何错误提示。Qt 5 使用 C++11 支持 Lambda 表达式，connect() 的时候如果函数名写错了就会在编译时报错，还有一点是 Lambda 表达式在需要的时候才定义，不需要声明，写起来比较简单。

Lambda 表达式可以理解为匿名函数，比如代码里有一些小函数，而这些函数一般只被调用一次（比如函数指针），这时就可以用 Lambda 表达式替代他们，这样代码看起来更简洁些，用起来也方便。

<!--more-->

## Lambda 语法简介
![](/img/qt/cpp-lambda.png)

1. Capture clause: 捕获子句
2. Parameter list: 参数列表 `可选`
3. Mutable specification `可选`
4. Exception specification `可选`
5. Return type: 返回类型 `可选`
6. Lambda Body

捕获子句的使用说明:

| 用法      | 说明                                 |
| ------- | ---------------------------------- |
| []      | 表明 Lambda body 不访问 `闭包` 前面已声明的任何变量 |
| [=]     | 以值的方式访问 `闭包` 前面已声明的变量              |
| [&]     | 以引用的方式访问 `闭包` 前面已声明变量              |
| [this]  | 访问类实例的 this 指针                     |
| [x, &y] | x 以传值形式捕获，y 以引用形式捕获                |
| [=, &z] | z 以引用形式捕获，其余变量以传值形式捕获              |

> * 对于 [=] 或 [&] 的形式，lambda 表达式可以直接使用 this 指针  
> * 闭包指的是一个拥有许多变量和绑定了这些变量的环境的表达式(通常是一个函数，Lambda 表达式就是一个闭包)，因而这些变量也是该表达式的一部分

下面列举一些 Lambda 表达式在 Qt 中的运用。

## 信号槽
```cpp
#include <QApplication>
#include <QDebug>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QPushButton *button = new QPushButton("点击");
    button->show();

    QObject::connect(button, &QPushButton::clicked, []() {
        qDebug() << "点击";
    });

    return app.exec();
}
```

## 信号槽(重载)

```cpp
#include <QApplication>
#include <QDebug>
#include <QComboBox>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QComboBox *comboBox = new QComboBox();
    comboBox->addItem("林冲");
    comboBox->addItem("鲁达");
    comboBox->addItem("武松");
    comboBox->show();

    QObject::connect(comboBox, &QComboBox::activated, []() {
        qDebug() << "Hello";
    });

    return app.exec();
}
```

编译报错: `No matching function for call to 'connect'`，原因是信号 QComboBox::activated() 有重载函数:

* void QComboBox::activated(int index)
* void QComboBox::activated(const QString &text)

在进行信号槽绑定时，如果有重载，需要对成员函数进行类型转换，可以使用 C++ 的 `static_cast` 类型转换(编译时进行语法检查)，也可以使用传统的 C 语言的强制类型转换(编译时不进行语法检查，运行时才检查)，或者 C++11 的 QOverload::of，C++14 的 qOverload:

```cpp
#include <QApplication>
#include <QDebug>
#include <QComboBox>
#include <QtGlobal>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QComboBox *comboBox = new QComboBox();
    comboBox->addItem("林冲");
    comboBox->addItem("鲁达");
    comboBox->addItem("武松");
    comboBox->show();

    QObject::connect(comboBox, static_cast<void (QComboBox::*)(int)>(&QComboBox::activated), [](int index) {
        qDebug() << index;
    });

    QObject::connect(comboBox, static_cast<void (QComboBox::*)(const QString &)>(&QComboBox::activated), [](const QString &text) {
        qDebug() << text;
    });

    QObject::connect(comboBox, QOverload<const QString &>::of(&QComboBox::activated), [](const QString &text) {
        qDebug() << text;
    });

    return app.exec();
}
```

> **Qt 文档: Selecting Overloaded Signals and Slots:**
>
> With the string-based syntax, parameter types are explicitly specified. As a result, the desired instance of an overloaded signal or slot is unambiguous.
> In contrast, with the functor-based syntax, an overloaded signal or slot must be casted to tell the compiler which instance to use.
> For example, QLCDNumber has three versions of the display() slot:
>
> 1. QLCDNumber::display(int)
> 2. QLCDNumber::display(double)
> 3. QLCDNumber::display(QString)
>
> To connect the int version to QSlider::valueChanged(), the two syntaxes are:
>
```cpp
auto slider = new QSlider(this);
auto lcd    = new QLCDNumber(this);

// String-based syntax
connect(slider, SIGNAL(valueChanged(int)), lcd, SLOT(display(int)));

// Functor-based syntax, first alternative
connect(slider, &QSlider::valueChanged, lcd, static_cast<void (QLCDNumber::*)(int)>(&QLCDNumber::display));

// Functor-based syntax, second alternative
void (QLCDNumber::*mySlot)(int) = &QLCDNumber::display; 
connect(slider, &QSlider::valueChanged, lcd, mySlot);

// Functor-based syntax, third alternative
connect(slider, &QSlider::valueChanged, lcd, QOverload<int>::of(&QLCDNumber::display));

// Functor-based syntax, fourth alternative (requires C++14)
connect(slider, &QSlider::valueChanged, lcd, qOverload<int>(&QLCDNumber::display));
```

## 排序

```cpp
#include <QDebug>
#include <QList>
#include <algorithm>

int main(int argc, char *argv[]) {
    QList<int> ns = QList<int>() << 1 << 3 << 5 << 4 << 2;
    
    // 传入 sort 需要的比较函数，进行降序排序
    std::sort(ns.begin(), ns.end(), [](int a, int b) -> bool {
        return a > b;
    });

    qDebug() << ns;

    return 0;
}
```

## 自定义函数参数为 Lambda 表达式
使用 `std::function<>` 声明 Lambda 表达式

```cpp
#include <QDebug>

// 第二个参数为一个 Lambda 表达式，其参数是 int，返回值为 int
int foo(int n, std::function<int (int)> process) {
    qDebug() << "Input is: " << n;
    return process(n);
}

int main(int argc, char *argv[]) {
    // 传入的 Lambda 为求阶乘
    int result = foo(5, [](int n) -> int {
        int factorial = 1;

        for (int i = 1; i <= n; ++i) {
            factorial *= i;
        }

        return factorial;
    });

    qDebug() << result;

    return 0;
}
```

输出:

```
Input is:  5
120
```

## 参考资料
* [C++11 新特性: Lambda 表达式](https://www.devbean.net/2012/05/cpp11-lambda/)
* [C++11 Lambda 表达式](http://www.cnblogs.com/zhuyp1015/archive/2012/04/08/2438176.html)
