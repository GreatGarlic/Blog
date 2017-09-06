---
title: 九宫格绘图
date: 2017-09-06 15:36:38
tags: QtBook
---

很多时候都会使用图片作为 widget 的背景，如果图片和 widget 一样大的话那就没什么好说的，背景效果和图片的效果看上去一样，可更多的时候我们会面临图片和 widget 不一样大，如果把图片简单的缩放到和 widget 一样大作为背景的话，背景常常会变形、有锯齿等，如下面的背景图大小为 128 x 108，要作为 300 x 200 大小的背景，直接缩放绘制的效果很不好，如若使用接下来将要介绍的九宫格绘图技术来绘制背景的话，效果正是我们期望的: 

* 左边是直接缩放绘制的效果，背景发虚，有锯齿，圆角被放大
* 右边是九宫格技术绘制的效果，圆角和背景的圆角一样

![](/img/qtbook/paint/Paint-Nine-Patch-Painter-bkg.png)

![](/img/qtbook/paint/Paint-Nine-Patch-Painter-1.png)<!--more-->

## 九宫格绘图原理

什么是九宫格呢？顾名思义，就是把一个方块分割成 9 个部分，如图所示：

![](/img/qtbook/paint/Paint-Nine-Patch-Painter-2.png)

九宫格绘图的原理就是把背景图分割成 9 个部分，绘制时:

* 4 个角的大小不变
* 左、右部分宽度不变，进行垂直拉伸或平铺绘制
* 上、下部分高度不变，进行水平拉伸或平铺绘制
* 中间部分进行拉伸或平铺绘制

这样终绘制出来的效果非常接近背景图效果。

## NinePatchPainter

了解九宫格绘制的原理后，接下来实现工具类 NinePatchPainter，在任意的矩形(widget 也是矩形的)内使用九宫格绘制，只需要简单的调用 `NinePatchPainter::paint(painter, rect)` 即可。

```cpp
// 文件名: NinePatchPainter.h

#ifndef NINEPATCHPAINTER_H
#define NINEPATCHPAINTER_H

class QRect;
class QMargins;
class QPainter;
class QPixmap;
class NinePatchPainterPrivate;

/**
 * @brief NinePatchPainter 用于九宫格的方式绘图，当背景图和需要绘制的范围不一样大时，能够最大限度的保证绘制出来的效果和背景图接近.
 *
 * 需要提供 QPixmap 的背景图和九宫格的 4 个变宽来创建 NinePatchPainter 对象，绘图的接口很简单，只有 2 个参数，QPainter 和 QRect，
 * 调用 NinePatchPainter.paint(painter, rect) 就使用了九宫格的方式绘图，不需要其他的操作.
 */
class NinePatchPainter {
public:
    /**
     * @brief 使用 pixmap, 九宫格的 4 个边宽，水平和垂直的缩放方式创建 NinePatchPainter 对象.
     *
     * @param background 背景图
     * @param left   左边宽
     * @param top    上边高
     * @param right  右边宽
     * @param bottom 下边高
     * @param horizontalStretch 水平方向是否使用拉伸绘制，默认为 true
     * @param verticalStretch   垂直方向是否使用拉伸绘制，默认为 true
     */
    NinePatchPainter(const QPixmap &background,
                     int left, int top, int right, int bottom,
                     bool horizontalStretch = true, bool verticalStretch = true);
    ~NinePatchPainter();

    /**
     * @brief 在 rect 中使用九宫格的方式进行绘图.
     */
    void paint(QPainter *painter, const QRect &rect) const;

private:
    NinePatchPainterPrivate *d;
};

#endif // NINEPATCHPAINTER_H
```

```cpp
// 文件名: NinePatchPainter.cpp

#include "NinePatchPainter.h"
#include <QPixmap>
#include <QList>
#include <QRect>
#include <QPainter>
#include <QPixmap>

/*-----------------------------------------------------------------------------|
 |                           NinePatchPainterPrivate                           |
 |----------------------------------------------------------------------------*/
class NinePatchPainterPrivate {
public:
    NinePatchPainterPrivate(const QPixmap &background,
                            int left, int top, int right, int bottom,
                            bool horizontalStretch, bool verticalStretch);

    // 根据九宫格 4 边的宽度把 rect 按九宫格分割为 9 个 rect: 左、左上、上、右上、右、右下、下、左下、中间
    QList<QRect> calculateNinePatchRects(const QRect &rect) const;

    // 对图片进行缩放
    QPixmap scalePixmap(const QPixmap &pixmap, const QSize &size) const;

public:
    int  left;   // 左边的宽
    int  top;    // 上边的宽
    int  right;  // 右边的宽
    int  bottom; // 下边的宽
    bool horizontalStretch; // 水平方向是否使用拉伸绘制
    bool verticalStretch;   // 垂直方向是否使用拉伸绘制

    QPixmap leftPixmap;        // 左边的子图
    QPixmap topLeftPixmap;     // 左上角的子图
    QPixmap topPixmap;         // 顶部的子图
    QPixmap topRightPixmap;    // 右上角的子图
    QPixmap rightPixmap;       // 右边的子图
    QPixmap bottomLeftPixmap;  // 左下角的子图
    QPixmap bottomPixmap;      // 底部的子图
    QPixmap bottomRightPixmap; // 右下角的子图
    QPixmap centerPixmap;      // 中间的子图
};

NinePatchPainterPrivate::NinePatchPainterPrivate(const QPixmap &background,
                                                 int left, int top, int right, int bottom,
                                                 bool horizontalStretch, bool verticalStretch)
    : left(left), top(top), right(right), bottom(bottom),
      horizontalStretch(horizontalStretch), verticalStretch(verticalStretch) {

    // 把 background 分割成 9 个子图，程序运行期间不会变，所以缓存起来
    QRect pixmapRect(0, 0, background.width(), background.height());
    QList<QRect> rects = calculateNinePatchRects(pixmapRect);

    leftPixmap        = background.copy(rects.at(0));
    topLeftPixmap     = background.copy(rects.at(1));
    topPixmap         = background.copy(rects.at(2));
    topRightPixmap    = background.copy(rects.at(3));
    rightPixmap       = background.copy(rects.at(4));
    bottomRightPixmap = background.copy(rects.at(5));
    bottomPixmap      = background.copy(rects.at(6));
    bottomLeftPixmap  = background.copy(rects.at(7));
    centerPixmap      = background.copy(rects.at(8));
}

QList<QRect> NinePatchPainterPrivate::calculateNinePatchRects(const QRect &rect) const {
    int x = rect.x();
    int y = rect.y();
    int cw = rect.width() - left - right;  // 中间部分的宽
    int ch = rect.height() - top - bottom; // 中间部分的高

    // 根据把 rect 分割成 9 个部分: 左、左上、上、右上、右、右下、下、左下、中间
    QRect leftRect(x, y + top, left, ch);
    QRect topLeftRect(x, y, left, top);
    QRect topRect(x + left, y, cw, top);
    QRect topRightRect(x + left + cw, y, right, top);
    QRect rightRect(x + left + cw, y + top, right, ch);
    QRect bottomRightRect(x + left + cw, y + top + ch, right, bottom);
    QRect bottomRect(x + left, y + top + ch, cw, bottom);
    QRect bottomLeftRect(x, y + top + ch, left, bottom);
    QRect centerRect(x + left, y + top, cw, ch);

    return QList<QRect>() << leftRect << topLeftRect
                          << topRect << topRightRect << rightRect
                          << bottomRightRect << bottomRect << bottomLeftRect
                          << centerRect;
}

QPixmap NinePatchPainterPrivate::scalePixmap(const QPixmap &pixmap, const QSize &size) const {
    // 缩放时忽略图片的高宽比，使用平滑缩放的效果
    return pixmap.scaled(size, Qt::IgnoreAspectRatio, Qt::SmoothTransformation);
}

/*-----------------------------------------------------------------------------|
 |                              NinePatchPainter                               |
 |----------------------------------------------------------------------------*/
NinePatchPainter::NinePatchPainter(const QPixmap &background,
                                   int left, int top, int right, int bottom,
                                   bool horizontalStretch, bool verticalStretch)
    : d(new NinePatchPainterPrivate(background, left, top, right, bottom, horizontalStretch, verticalStretch)) {
}

NinePatchPainter::~NinePatchPainter() {
    delete d;
}


void NinePatchPainter::paint(QPainter *painter, const QRect &rect) const {
    // 把要绘制的 Rect 分割成 9 个部分，上，右，下，左 4 边的宽和背景图的一样
    QList<QRect> rects = d->calculateNinePatchRects(rect);

    QRect leftRect        = rects.at(0);
    QRect topLeftRect     = rects.at(1);
    QRect topRect         = rects.at(2);
    QRect topRightRect    = rects.at(3);
    QRect rightRect       = rects.at(4);
    QRect bottomRightRect = rects.at(5);
    QRect bottomRect      = rects.at(6);
    QRect bottomLeftRect  = rects.at(7);
    QRect centerRect      = rects.at(8);

    // 绘制 4 个角
    painter->drawPixmap(topLeftRect,     d->topLeftPixmap);
    painter->drawPixmap(topRightRect,    d->topRightPixmap);
    painter->drawPixmap(bottomRightRect, d->bottomRightPixmap);
    painter->drawPixmap(bottomLeftRect,  d->bottomLeftPixmap);

    // 绘制左、右边
    if (d->horizontalStretch) {
        // 水平拉伸
        painter->drawPixmap(leftRect,  d->scalePixmap(d->leftPixmap,  leftRect.size()));
        painter->drawPixmap(rightRect, d->scalePixmap(d->rightPixmap, rightRect.size()));
    } else {
        // 水平平铺
        painter->drawTiledPixmap(leftRect,  d->leftPixmap);
        painter->drawTiledPixmap(rightRect, d->rightPixmap);
    }

    // 绘制上、下边
    if (d->verticalStretch) {
        // 垂直拉伸
        painter->drawPixmap(topRect,    d->scalePixmap(d->topPixmap,    topRect.size()));
        painter->drawPixmap(bottomRect, d->scalePixmap(d->bottomPixmap, bottomRect.size()));
    } else {
        // 垂直平铺
        painter->drawTiledPixmap(topRect,    d->topPixmap);
        painter->drawTiledPixmap(bottomRect, d->bottomPixmap);
    }

    int pmw = d->centerPixmap.width();
    int pmh = d->centerPixmap.height();
    int crw = centerRect.width();
    int crh = centerRect.height();

    // 绘制中间部分(最简单办法就是中间部分都进行拉伸)
    if (d->horizontalStretch && d->verticalStretch) {
        // 水平和垂直都拉伸
        painter->drawPixmap(centerRect, d->scalePixmap(d->centerPixmap, centerRect.size()));
    } else if (d->horizontalStretch && !d->verticalStretch) {
        // 水平拉伸，垂直平铺
        if (crh % pmh != 0) {
            pmh = ((float)crh) / (crh/pmh+1);
        }
        QSize size(crw, pmh);
        QPixmap centerPixmap = d->scalePixmap(d->centerPixmap, size);
        painter->drawTiledPixmap(centerRect, centerPixmap);
    } else if (!d->horizontalStretch && d->verticalStretch) {
        // 水平平铺，垂直拉伸
        if (crw % pmw != 0) {
            pmw = ((float)crw) / (crw/pmw+1);
        }
        QSize size(pmw, crh);
        QPixmap centerPixmap = d->scalePixmap(d->centerPixmap, size);
        painter->drawTiledPixmap(centerRect, centerPixmap);
    } else {
        // 水平和垂直都平铺
        painter->drawTiledPixmap(centerRect, d->centerPixmap);
    }
}
```

九宫格的核心是矩形分割和不同的矩形区域内使用拉伸还是平铺绘图，矩形分隔并不难，只是写程序时凭想象和记忆计算坐标容易出错，可以先在纸上画来写上坐标，然后再写程序就会很直观了。绘制时的难点在绘制中间部分，例如水平拉伸、垂直平铺应该怎么理解呢？分两步，一是先水平拉伸图片，拉伸的矩形高度为图片的高度，宽度为中间部分的宽度(想象一下图片高度不变，水平方向给拉长了)，然后使用 drawTiledPixap() 绘图，这样垂直方向绘制时就时平铺的了。

## 案例展示

下面就用一个简单的案例展示 NinePatchPainter 的威力，重点关注 paintEvent() 函数，效果图请参考文章开头出的效果图.

```cpp
// 文件名: Widget.h

#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include <QPixmap>

class NinePatchPainter;

class Widget : public QWidget {
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

protected:
    void paintEvent(QPaintEvent *event) Q_DECL_OVERRIDE;

private:
    QPixmap pixmap;
    NinePatchPainter *ninePatchPainter;
};

#endif // WIDGET_H
```

```cpp
// 文件名: Widget.cpp

#include "Widget.h"
#include "NinePatchPainter.h"

#include <QPixmap>
#include <QPainter>

Widget::Widget(QWidget *parent) : QWidget(parent) {
    pixmap.load(":/rounded-rectangle.png"); // 加载背景图
    ninePatchPainter = new NinePatchPainter(pixmap, 25, 25, 25, 25);
}

Widget::~Widget() {
    delete ninePatchPainter;
}

void Widget::paintEvent(QPaintEvent *) {
    QPainter painter(this);

    painter.drawPixmap(20, 20, 300, 200, pixmap); // 普通的拉伸绘制
    ninePatchPainter->paint(&painter, QRect(340, 20, 300, 200)); // 九宫格绘制
}
```

```cpp
// 文件名: main.cpp

#include "Widget.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    Widget w;
    w.setWindowTitle("九宫格绘图");
    w.resize(660, 240);
    w.show();

    return a.exec();
}
```

## 思考

有了背景图，怎么确定各边的宽高是多少像素才合适呢？一般是各边的大小不小于对应角的大小，这样 4 个角绘制出来才不会变形。可以参考一下 Border Image 一节 <http://qtdebug.com/qtbook-qss-border-image>，这样就能理解的更深刻了。

## 作业

左图为按钮的背景图，实现右图所示的各种大小的按钮效果。

| 圆角按钮背景                                   | 圆角按钮效果                                   |
| ---------------------------------------- | ---------------------------------------- |
| ![](/img/qtbook/qss/QSS-BorderImage-Round-Button.png) | ![](/img/qtbook/qss/QSS-BorderImage-Buttons.png) |

