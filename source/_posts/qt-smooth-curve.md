---
title: Qt 绘制平滑曲线
date: 2016-09-29 14:26:21
tags: Qt
---
Qt 中可以使用 `QPainterPath::cubicTo()` 函数绘制如下的平滑曲线

![](/img/qt/smooth-curve-1.png)

<!--more-->

绘制平滑曲线的关键是控制点的计算，sp 为线段的起始点，ep 为线段的终点，c1，c2 为贝塞尔曲线的控制点，其坐标计算如下

![](/img/qt/smooth-curve-2.png)

下面就用个简单的程序介绍怎么绘制平滑曲线，主要部分为 for 循环里控制点 c1, c2 的计算，注意不能把它们的顺序弄反了哦，否则生成的曲线很奇怪的。

```cpp
#include "Widget.h"
#include "ui_Widget.h"

#include <QPainterPath>
#include <QPainter>
#include <QList>

Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget) {
    ui->setupUi(this);
}

Widget::~Widget() {
    delete ui;
}

void Widget::paintEvent(QPaintEvent *) {
    // 曲线上的点
    static QList<QPointF> points = QList<QPointF>() << QPointF(0, 0) << QPointF(100, 100) << QPointF(200, -100)
                                                    << QPointF(300, 100) << QPointF(330, -80) << QPointF(350, -70);
    QPainterPath path(points[0]);

    for (int i = 0; i < points.size() - 1; ++i) {
        // 控制点的 x 坐标为 sp 与 ep 的 x 坐标和的一半
        // 第一个控制点 c1 的 y 坐标为起始点 sp 的 y 坐标
        // 第二个控制点 c2 的 y 坐标为结束点 ep 的 y 坐标
        QPointF sp = points[i];
        QPointF ep = points[i+1];
        QPointF c1 = QPointF((sp.x() + ep.x()) / 2, sp.y());
        QPointF c2 = QPointF((sp.x() + ep.x()) / 2, ep.y());

        path.cubicTo(c1, c2, ep);
    }

    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing, true);
    painter.setPen(QPen(Qt::black, 2));

    // 绘制 path
    painter.translate(40, 130);
    painter.drawPath(path);

    // 绘制曲线上的点
    painter.setBrush(Qt::gray);
    for (int i = 0; i < points.size(); ++i) {
        painter.drawEllipse(points[i], 4, 4);
    }
}
```
