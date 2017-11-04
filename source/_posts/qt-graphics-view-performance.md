---
title: 测试 Graphics View 的效率
date: 2017-11-04 12:00:14
tags: Qt
---

Qt 说 Graphics View Framework 效率很高，到底有多高呢？来进行一个简单的测试，向  scene 中添加 10万，50万，100万个 items(修改程序中的 rowCount 和 colCount 即可)，进行缩放、旋转看看效率怎么样。

> 在可视区域内的 items 少的话，不管 scene 里有多少个 items，10万个和 100万个的区别不大，效率都是非常高的，但可视区内 items 越多的话，越多效率越低。Qt 使用 [Binary-Space-Partitioning](https://en.wikipedia.org/wiki/Binary_space_partitioning) 算法管理 items，能够快速的找出可视区内的 items 进行绘制(100万个 items 中可能只需要绘制 100 个 items)，不会绘制所有的 items，绘制操作是非常耗时的，这也就是为什么影响效率最大的因素是可视区内的 items。
>
> 实际项目中添加 10万个 items 的效率和此处测试的 10万个 items 的效率是有些微区别的，绘制的消耗由其 `paint()` 函数决定。<!--more-->

```cpp
// 文件名: Widget.h

#ifndef WIDGET_H
#define WIDGET_H

#include <QGraphicsView>

class Widget : public QGraphicsView {
    Q_OBJECT

public:
    Widget(QWidget *parent = 0);
    ~Widget();

protected:
    void keyPressEvent(QKeyEvent *event) Q_DECL_OVERRIDE;
};

#endif // WIDGET_H
```

```cpp
// 文件名: Widget.cpp

#include "Widget.h"
#include <QGraphicsScene>
#include <QGraphicsRectItem>
#include <QKeyEvent>

Widget::Widget(QWidget *parent) : QGraphicsView(parent) {
    // 随机定义 100 个颜色
    QColor *colors = new QColor[100];
    for (int i = 0; i < 100; ++i) {
        colors[i] = QColor(qrand() % 256, qrand() % 256, qrand() % 256, qrand() % 256);
    }

    QGraphicsScene *scene = new QGraphicsScene(this);
    QGraphicsItem   *item = NULL;

    // 创建 rowCount * colCount 个 items
    const int rowCount = 100;
    const int colCount = 1000;
    for (int row = 0; row < rowCount; row += 1) {
        for (int col = 0; col < colCount; col += 1) {
            item = scene->addRect(col << 5, row << 5, 24, 24, QPen(Qt::darkGray), QBrush(colors[qrand() % 100]));
            item->setFlags(QGraphicsItem::ItemIsMovable | QGraphicsItem::ItemIsSelectable);
        }
    }

    this->setScene(scene);
    this->setDragMode(QGraphicsView::RubberBandDrag);
    this->setRenderHint(QPainter::Antialiasing);

    delete[] colors;
}

Widget::~Widget() {

}

void Widget::keyPressEvent(QKeyEvent *event) {
    // A 左旋
    // D 右旋
    // W 放大
    // S 缩小
    switch (event->key()) {
    case Qt::Key_A:
        rotate(-3);
        break;
    case Qt::Key_D:
        rotate(3);
        break;
    case Qt::Key_W:
        scale(1.1, 1.1);
        break;
    case Qt::Key_S:
        scale(1/1.1, 1/1.1);
        break;
    }

    event->accept();
}
```

```cpp
// 文件名: main.cpp

#include "Widget.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    Widget w;
    w.show();

    return a.exec();
}
```

[Binary-Space-Partitioning](https://en.wikipedia.org/wiki/Binary_space_partitioning) 简介:

> In computer science, binary space partitioning (BSP) is a method for recursively subdividing a space into convex sets by hyperplanes. This subdivision gives rise to a representation of objects within the space by means of a tree data structure known as a BSP tree.
>
> Binary space partitioning was developed in the context of 3D computer graphics,[1][2] where the structure of a BSP tree allows spatial information about the objects in a scene that is useful in rendering, such as their ordering from front-to-back with respect to a viewer at a given location, to be accessed rapidly. Other applications include performing geometrical operations with shapes (constructive solid geometry) in CAD,[3] collision detection in robotics and 3-D video games, ray tracing and other computer applications that involve handling of complex spatial scenes.