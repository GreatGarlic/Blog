---
title: Qt 创建圆角、无边框、有阴影、可拖动的窗口
date: 2016-09-07 22:52:44
tags: Qt
---

程序窗口的边框，标题栏等是系统管理的，Qt 不能对其进行定制，为了实现定制的边框、标题栏、关闭按钮等，需要把系统默认的边框、标题栏去掉，然后使用 Widget 来模拟它们。这里介绍使用 `QSS + QGraphicsDropShadowEffect` 来创建圆角、无边框、有阴影、可拖动的窗口。

**核心技术要点:**

* 启用 QSS: `setAttribute(Qt::WA_StyledBackground, true)`

    > 我们继承 QWidget 实现的 Widget 默认是不启用 QSS 的，为了启用 QSS，需要调用 `setAttribute(Qt::WA_StyledBackground, true)`
* 使用 `border-radius` 创建圆角效果

    > 顶级窗口有些 QSS 不生效，例如 `border-radius`，所以把要显示圆角的 Widget 上放在另一个顶级 Widget 中，变为非顶级窗口
* 顶级窗口需要去掉边框，背景设置为透明
    * 去掉边框: `setWindowFlags(Qt::FramelessWindowHint);`
    * 背景透明: `setAttribute(Qt::WA_TranslucentBackground);`
* 使用鼠标事件实现拖动
* 使用 `QGraphicsDropShadowEffect` 创建阴影

    > 很遗憾，QSS 不支持阴影

<!--more-->

**使用方法:**

* `FramelessWindow *window = new FramelessWindow(yourWidget)` 即可

**效果如图:**  
![](/img/qt/frameless-window.png)

## main.cpp
```cpp
#include "FramelessWindow.h"

#include <QDebug>
#include <QApplication>
#include <QWidget>
#include <QLabel>
#include <QPushButton>
#include <QTextEdit>
#include <QVBoxLayout>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    // 创建包含主要控件的 Widget
    QPushButton *quitButton = new QPushButton("退出");
    QVBoxLayout *layout = new QVBoxLayout();
    layout->addWidget(new QLabel("按住我拖动也可以拖动窗口的哦"));
    layout->addWidget(new QTextEdit());
    layout->addWidget(quitButton);

    QWidget *contentWidget = new QWidget();
    contentWidget->setLayout(layout);
    contentWidget->setObjectName("contentWidget");
    contentWidget->setStyleSheet("#contentWidget{background: lightgray; border-radius: 4px;}" // 定制圆角
                                 ".QLabel{background: gray;}.QTextEdit{background: white;}");

    QObject::connect(quitButton, &QPushButton::clicked, [&app] {
        app.quit();
    });

    // 创建无边框、有阴影、可拖动的窗口
    FramelessWindow *window = new FramelessWindow(contentWidget);
    window->resize(300, 400);
    window->show();

    return app.exec();
}
```

## FramelessWindow.h
```cpp
#ifndef FRAMELESSWINDOW_H
#define FRAMELESSWINDOW_H

#include <QWidget>

struct FramelessWindowPrivate;

class FramelessWindow : public QWidget {
    Q_OBJECT
public:
    explicit FramelessWindow(QWidget *contentWidget, QWidget *parent = 0);
    ~FramelessWindow();

protected:
    void mousePressEvent(QMouseEvent *e) Q_DECL_OVERRIDE;
    void mouseReleaseEvent(QMouseEvent *e) Q_DECL_OVERRIDE;
    void mouseMoveEvent(QMouseEvent *e) Q_DECL_OVERRIDE;

private:
    FramelessWindowPrivate *d;
};

#endif // FRAMELESSWINDOW_H
```

## FramelessWindow.cpp
```cpp
#include "FramelessWindow.h"

#include <QMouseEvent>
#include <QGridLayout>
#include <QGraphicsDropShadowEffect>

struct FramelessWindowPrivate {
    FramelessWindowPrivate(QWidget *contentWidget) : contentWidget(contentWidget) {}

    QWidget *contentWidget;
    QPoint mousePressedPosition; // 鼠标按下时的坐标
    QPoint windowPositionAsDrag; // 鼠标按小时窗口左上角的坐标
};

FramelessWindow::FramelessWindow(QWidget *contentWidget, QWidget *parent) : QWidget(parent) {
    setWindowFlags(Qt::FramelessWindowHint);    // 去掉边框
    setAttribute(Qt::WA_TranslucentBackground); // 背景透明

    d = new FramelessWindowPrivate(contentWidget);

    // 添加阴影
    QGraphicsDropShadowEffect *shadowEffect = new QGraphicsDropShadowEffect(contentWidget);
    shadowEffect->setColor(Qt::lightGray);
    shadowEffect->setBlurRadius(4); // 阴影的大小
    shadowEffect->setOffset(0, 0);
    contentWidget->setGraphicsEffect(shadowEffect);

    // 添加到窗口中
    QGridLayout *lo = new QGridLayout();
    lo->addWidget(contentWidget, 0, 0);
    lo->setContentsMargins(4, 4, 4, 4); // 注意和阴影大小的协调
    setLayout(lo);
}

FramelessWindow::~FramelessWindow() {
    delete d;
}

void FramelessWindow::mousePressEvent(QMouseEvent *e) {
    // 记录鼠标按下时全局的位置和窗口左上角的位置
    d->mousePressedPosition = e->globalPos();
    d->windowPositionAsDrag = pos();
}

void FramelessWindow::mouseReleaseEvent(QMouseEvent *e) {
    Q_UNUSED(e)
    // 鼠标放开始设置鼠标按下的位置为 null，表示鼠标没有被按下
    d->mousePressedPosition = QPoint();
}

void FramelessWindow::mouseMoveEvent(QMouseEvent *e) {
    if (!d->mousePressedPosition.isNull()) {
        // 鼠标按下并且移动时，移动窗口, 相对于鼠标按下时的位置计算，是为了防止误差累积
        QPoint delta = e->globalPos() - d->mousePressedPosition;
        move(d->windowPositionAsDrag + delta);
    }
}
```

## 思考
**还可以使用其他方式实现上面的功能，并且功能也不够丰富，思考下面的问题:**

* 使用其他方式实现圆角、阴影，例如:
    * 绘图
        * 绘制圆角矩形并且实现阴影的算法
        * 使用一个圆角带阴影图片，利用九宫格技术绘制(border-image 的原理)
    * QSS 的 `border-image`
* 拖动调整无边框窗口的大小
* 添加标题栏
* 添加最小化、最大化、关闭按钮


