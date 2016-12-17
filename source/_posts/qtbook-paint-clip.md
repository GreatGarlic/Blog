---
title: Clip 实现复杂绘图效果
date: 2016-12-17 10:06:46
tags: QtBook
---

经常会遇到有人问，用 Pixmap 绘制图像已经知道了，但是怎么绘制一个圆的图像呢？就像 QQ 头像那样，即使上传的头像是一个矩形的，但是显示出来的效果是圆形的，也就是说，在圆的范围内绘制图像，图像超过圆范围的部分不绘制，就像下图的效果

![](/img/qtbook/paint/Paint-Clip.png)

<!--more-->

如果仔细看过 QPainter 的 API，就会注意到有几个以 setClip 开头的函数，例如 setClipPath

> Enables clipping, and sets the clip path for the painter to the given path, with the clip operation.
>
> Note that the clip path is specified in logical (painter) coordinates.

clip 的作用是通过对元素进行剪切来控制元素的可显示区域，也就是在 clip 指定的范围内的内容才能显示出来，正是使用任意形状的 QPainterPath 作为 QPainter 的 clip，就能绘制出我们想要的效果，例如绘制上图显示的圆形和星形的图片，clip 的使用请参考下面的代码。

```cpp
// ClipWidget.h
#ifndef CLIPWIDGET_H
#define CLIPWIDGET_H

#include <QWidget>

class ClipWidget : public QWidget {
    Q_OBJECT

public:
    explicit ClipWidget(QWidget *parent = 0);
    ~ClipWidget();

protected:
    void paintEvent(QPaintEvent *event) Q_DECL_OVERRIDE;
};

#endif // CLIPWIDGET_H
```

```cpp
// ClipWidget.cpp
#include "ClipWidget.h"

#include <QPainter>
#include <QPixmap>
#include <QPainterPath>

ClipWidget::ClipWidget(QWidget *parent) : QWidget(parent) {
    resize(300, 150);
}

ClipWidget::~ClipWidget() {
}

void ClipWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing, true);
    painter.translate(width()/2, height()/2);
    painter.setPen(QPen(Qt::gray, 4));

    QPixmap pixmap("/Users/Biao/Pictures/dog.png");

    // [1.1] 创建圆的 path
    QPainterPath circlePath;
    circlePath.addEllipse(-50, -50, 100, 100);

    painter.translate(-60, 0);
    painter.setClipPath(circlePath); // [1.2] 指定 clip 为圆的 path
    painter.drawPixmap(circlePath.boundingRect().toRect(), pixmap); // [1.3] 绘制出来的 pixmap 只会显示在圆中的部分
    painter.drawPath(circlePath);

    // 同上
    // [2.1] 创建星形的 path
    QPainterPath starPath;
    starPath.moveTo(0, -50);
    starPath.lineTo(20, -20);
    starPath.lineTo(50, 0);
    starPath.lineTo(20, 20);
    starPath.lineTo(0, 50);
    starPath.lineTo(-20, 20);
    starPath.lineTo(-50, 0);
    starPath.lineTo(-20, -20);
    starPath.closeSubpath();

    painter.translate(120, 0);
    painter.setClipPath(starPath); // [2.2] 指定 clip 为星形的 path
    painter.drawPixmap(starPath.boundingRect().toRect(), pixmap); // [2.3] 绘制出来的 pixmap 只会显示在星形中的部分
    painter.drawPath(starPath);
}
```

```cpp
// main.cpp
#include "ClipWidget.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    ClipWidget w;
    w.show();
    return a.exec();
}
```

虽然也能够使用 QRegion 作为 clip，但是把 region 设置为圆形时，效果非常的差，例如全是锯齿，所以如果是复杂 clip 的话，推荐使用 QPainterPath 作为 clip，而且 QRegion 的表现形式也很有限，只能是矩形，椭圆或者其进行相交的结果，QPainterPath 则可以是任意的形状，能够发挥你的想象力，实现各种效果，思考一下，像下面的这些效果怎么实现？

![](/img/qtbook/paint/Paint-Clip-1.jpg)

![](/img/qtbook/paint/Paint-Clip-2.jpg)