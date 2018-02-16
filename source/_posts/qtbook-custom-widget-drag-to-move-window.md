---
title: 按下鼠标移动窗口
date: 2017-11-24 10:14:58
tags: QtBook
---

QWidget 已经实现了在标题栏按下鼠标移动窗口的功能，但是当实现无边框窗口时，因为没有了标题栏，移动窗口的功能就需要我们自己实现了，不过也不复杂，主要是处理鼠标的按下、移动、松开三个事件:

* 按下鼠标：记录此时鼠标的全局坐标和窗口左上角的坐标，并且设置鼠标为按下状态
* 移动鼠标：鼠标按下时移动鼠标，计算此时鼠标和鼠标按下时的位移差，加上按下鼠标时窗口左上角的坐标得到窗口新的坐标，移动窗口到此坐标
* 松开鼠标：设置鼠标为未按下状态<!--more-->

实现的代码非常简单，甚至都不需要过多解释，唯一需要提示一下的是没有使用专门的一个变量来标记鼠标是否按下，而是复用了 mousePressedPosition，鼠标放开始时设置其为空，当其值不为空时表示鼠标被按下。

```cpp
#ifndef WINDOW_H
#define WINDOW_H

#include <QWidget>

class Window : public QWidget {
    Q_OBJECT

public:
    Window(QWidget *parent = 0);
    ~Window();

protected:
    void mousePressEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mouseReleaseEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mouseMoveEvent(QMouseEvent *event) Q_DECL_OVERRIDE;

private:
    QPoint windowPositionBeforeMoving; // 移动窗口前窗口左上角的坐标
    QPoint mousePressedPosition;       // 按下鼠标时鼠标的全局坐标
};

#endif // WINDOW_H
```

```cpp
#include "Window.h"
#include <QMouseEvent>

Window::Window(QWidget *parent) : QWidget(parent) {
}

Window::~Window() {

}

// 鼠标按下时记录此时鼠标的全局坐标和窗口左上角的坐标
void Window::mousePressEvent(QMouseEvent *event) {
    mousePressedPosition = event->globalPos();
    windowPositionBeforeMoving = frameGeometry().topLeft();
}

// 鼠标放开时设置 mousePressedPosition 为空坐标
void Window::mouseReleaseEvent(QMouseEvent *) {
    mousePressedPosition = QPoint();
}

// 鼠标移动时如果 mousePressedPosition 不为空，则说明需要移动窗口
// 鼠标移动的位移差，就是窗口移动的位移差
void Window::mouseMoveEvent(QMouseEvent *event) {
    if (!mousePressedPosition.isNull()) {
        QPoint delta = event->globalPos() - mousePressedPosition;
        QPoint newPosition = windowPositionBeforeMoving + delta;
        move(newPosition);
    }
}
```

```cpp
#include "Window.h"
#include <QApplication>
#include <QShortcut>
#include <QKeySequence>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    Window w;
    w.setWindowFlags(Qt::FramelessWindowHint);
    w.show();

    // 退出窗口的快捷键
    QShortcut *shortcut = new QShortcut(QKeySequence(Qt::CTRL + Qt::Key_C), &w);
    QObject::connect(shortcut, &QShortcut::activated, [] {
        qApp->quit();
    });

    return app.exec();
}
```

> 由于无边框窗口没有了标题栏，关闭按钮就不能使用了，所以 `main()` 函数中介绍了个小技巧，使用快捷键退出程序。