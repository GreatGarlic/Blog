---
title: Qt 创建圆角、无边框、有阴影、可拖动的窗口
date: 2016-09-07 22:52:44
tags: Qt
---

程序窗口的边框，标题栏等是系统管理的，Qt 不能对其进行定制，为了实现定制的边框、标题栏、关闭按钮等，需要把系统默认的边框、标题栏去掉，然后使用 Widget 来模拟它们。这里介绍使用 `QSS + QGraphicsDropShadowEffect` 来创建圆角、无边框、有阴影、可拖动的窗口。

**核心技术要点:**

* 我们继承 QWidget 实现的 Widget 默认是不启用 QSS 的，启用 QSS 需要调用 `setAttribute(Qt::WA_StyledBackground, true)`
* 使用 `border-radius` 创建圆角效果。但是顶级窗口有些 QSS 不生效，例如 `border-radius`，所以把要圆角显示的 Widget 上放在另一个顶级 Widget 中，变为非顶级 Widget
* 顶底窗口需要设置背景为透明，去掉边框
    * 去掉边框: `setWindowFlags(Qt::FramelessWindowHint);`
    * 背景透明: `setAttribute(Qt::WA_TranslucentBackground);`
* 使用鼠标事件实现拖动
* 使用 QGraphicsDropShadowEffect 创建阴影

**使用方法:**

* `FramelessWidget *window = new FramelessWidget(yourWidget)` 即可

**效果如图:**  
![](/img/qt/frameless-widget.png)

<!--more-->

## main.cpp
```cpp
#include "FramelessWidget.h"

#include <QApplication>
#include <QWidget>
#include <QLabel>
#include <QPushButton>
#include <QTextEdit>
#include <QVBoxLayout>
#include <QDebug>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    // 创建包含主要控件的 Widget
    QPushButton *quitButton = new QPushButton("退出");
    QVBoxLayout *layout = new QVBoxLayout();
    layout->addWidget(new QLabel("按住我拖动也可以拖动窗口的哦"));
    layout->addWidget(new QTextEdit());
    layout->addWidget(quitButton);

    QWidget *centralWidget = new QWidget();
    centralWidget->setLayout(layout);
    centralWidget->setStyleSheet("#centralWidget{background: lightgray; border: 1px solid lightgray;}"
                                 ".QLabel{background: gray;}.QTextEdit{background: white;}");

    QObject::connect(quitButton, &QPushButton::clicked, [&app] {
        app.quit();
    });

    // 创建无边框、圆角、有阴影、可拖动的窗口
    FramelessWidget *window = new FramelessWidget(centralWidget);
    window->resize(300, 400);
    window->show();

    return app.exec();
}
```

## FramelessWidget.h
```cpp
#ifndef FRAMELESSWIDGET_H
#define FRAMELESSWIDGET_H

#include <QWidget>

class FramelessWidget : public QWidget {
    Q_OBJECT
public:
    explicit FramelessWidget(QWidget *centralWidget, QWidget *parent = 0);

protected:
    void mousePressEvent(QMouseEvent *e) Q_DECL_OVERRIDE;
    void mouseReleaseEvent(QMouseEvent *e) Q_DECL_OVERRIDE;
    void mouseMoveEvent(QMouseEvent *e) Q_DECL_OVERRIDE;

private:
    QPoint pressedMousePosition;      // 鼠标按下时的坐标
    QPoint topLeftPositionBeforeDrag; // 鼠标按小时窗口左上角的坐标
};

#endif // FRAMELESSWIDGET_H
```

## FramelessWidget.cpp
```cpp
#include "FramelessWidget.h"
#include <QMouseEvent>
#include <QGridLayout>
#include <QGraphicsDropShadowEffect>

FramelessWidget::FramelessWidget(QWidget *centralWidget, QWidget *parent) : QWidget(parent) {
    // 添加阴影
    QGraphicsDropShadowEffect *shadowEffect = new QGraphicsDropShadowEffect(centralWidget);
    shadowEffect->setColor(Qt::lightGray);
    shadowEffect->setBlurRadius(4); // 阴影的大小
    shadowEffect->setOffset(0, 0);
    centralWidget->setGraphicsEffect(shadowEffect);
    centralWidget->setAttribute(Qt::WA_StyledBackground, true); // 启用 QSS
    centralWidget->setObjectName("centralWidget");
    centralWidget->setStyleSheet(centralWidget->styleSheet() + "#centralWidget {border-radius: 4px}"); // 设置圆角

    QGridLayout *lo = new QGridLayout();
    lo->addWidget(centralWidget, 0, 0);
    lo->setContentsMargins(4, 4, 4, 4);
    setLayout(lo);

    setWindowFlags(Qt::FramelessWindowHint);     // 去掉边框
    setAttribute(Qt::WA_TranslucentBackground);  // 背景透明
}

void FramelessWidget::mousePressEvent(QMouseEvent *e) {
    // 记录鼠标按下时全局的位置和窗口左上角的位置
    pressedMousePosition = e->globalPos();
    topLeftPositionBeforeDrag = frameGeometry().topLeft();
}

void FramelessWidget::mouseReleaseEvent(QMouseEvent *e) {
    Q_UNUSED(e)
    // 鼠标放开始设置鼠标按下的位置为 null，表示鼠标没有被按下
    pressedMousePosition = QPoint();
}

void FramelessWidget::mouseMoveEvent(QMouseEvent *e) {
    // 鼠标按下并且移动时，移动窗口
    if (!pressedMousePosition.isNull()) {
        QPoint delta = e->globalPos() - pressedMousePosition;
        move(topLeftPositionBeforeDrag + delta); // 移动窗口
    }
}
```

## 思考
* 实现圆角、带阴影的其它方式:
    * 绘图
        * 绘制圆角矩形并且实现阴影的算法
        * 使用一个圆角带阴影图片，利用九宫格技术绘制(border-image 的原理)
    * QSS 的 `border-image`
* 怎么实现拖动改变无边框窗口的大小
