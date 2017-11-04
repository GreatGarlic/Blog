---
title: 测试 Widget 的效率
date: 2017-11-04 16:06:20
tags: Qt
---

Widget 的效率怎么样，来进行一个简单的测试，添加 1千，1万，2万个，……，10万个 QPushButton(修改程序中的 buttonsCount 即可)，看看程序的创建好按钮，点击按钮执行槽函数，程序退出效果怎么样。

>添加 1千个，1万个按钮的时候窗口显示的速度非常快。
>添加 2万个的时候就需要几秒窗口才显示出来。
>添加的越多窗口显示需要的时间越长，添加 10万个需要等很久。
>按钮越多，程序退出的时间就越长，不过即使是 10万个按钮，退出也就是多了几秒，因为释放的内存多，这倒是没什么。
>当窗口显示出来后，不管添加了多少个按钮，点击按钮，它的槽函数都是瞬间就被执行。
>在实际应用中，添加上百个 widgets 在窗口上的见过，但有谁会添加上万个 widgets 到窗口上？不担心被产品经理揍的可以试试！<!--more-->

```cpp
// 文件名: Widget.h

#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

class Widget : public QWidget {
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

private slots:
    void buttonClicked();
};

#endif // WIDGET_H
```

```cpp
// 文件名: Widget.cpp

#include "Widget.h"
#include <QGridLayout>
#include <QPushButton>
#include <QScrollArea>
#include <QDebug>

Widget::Widget(QWidget *parent) : QWidget(parent) {
    const int     buttonsCount = 1000; // 按钮的数量
    QGridLayout *buttonsLayout = new QGridLayout();

    for (int i = 0; i < buttonsCount; ++i) {
        // 每行显示 10 个
        int row = i / 10;
        int col = i % 10;
        QPushButton *button = new QPushButton(QString("%1-%2").arg(row+1).arg(col+1));
        buttonsLayout->addWidget(button, row, col);

        connect(button, SIGNAL(clicked()), this, SLOT(buttonClicked()));
    }

    // 把按钮全添加到 buttonsContainer 中，然后把 buttonsContainer 再加到 scrollArea 上
    QWidget *buttonsContainer = new QWidget();
    buttonsContainer->setLayout(buttonsLayout);

    QScrollArea *scrollArea = new QScrollArea(this);
    scrollArea->setWidget(buttonsContainer);

    QGridLayout *mainLayout = new QGridLayout();
    mainLayout->addWidget(scrollArea);
    this->setLayout(mainLayout);
}

Widget::~Widget() {
}

void Widget::buttonClicked() {
    QPushButton *button = qobject_cast<QPushButton*>(sender());
    qDebug() << button->text();
}
```

```cpp
文件名: main.cpp

#include "Widget.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    Widget w;
    w.show();

    return a.exec();
}
```

