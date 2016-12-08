---
title: 贝塞尔曲线
date: 2016-12-08 22:30:20
tags: Qt-Book
---

QPainterPath 可以用来画贝塞尔曲线，什么是贝塞尔曲线呢？开始学的时候，经常听到贝塞尔曲线，但一直不知道是什么东西，很神秘的样子，据说很复杂，一直没敢学，人类对陌生的东西总是有恐惧感，这一部分就来揭开贝塞尔曲线神秘的面纱（大部分内容都来自于网络）。

> 贝塞尔曲线（The Bézier Curves），是一种在计算机图形学中相当重要的参数曲线（3D的称为曲面）。贝塞尔曲线于1962年，由法国工程师皮埃尔·贝塞尔（Pierre Bézier）所发表，他运用贝塞尔曲线来为汽车的主体进行设计。
>
> 一般的矢量图形软件通过它来精确画出曲线，贝塞尔曲线由线段与节点组成，节点是可拖动的支点，线段像可伸缩的皮筋，我们在绘图工具上看到的钢笔工具就是来做这种矢量曲线的。贝塞尔曲线是计算机图形学中相当重要的参数曲线，在一些比较成熟的位图软件中也有贝塞尔曲线工具，如PhotoShop等。<!--more-->

贝塞尔曲线还是很抽象的，如果不是看了下面的这些动态图，演示了贝塞尔曲线的生成过程，估计仍然很难明白贝塞尔曲线是什么样的，控制点是什么，有什么用。

塞尔曲线的通用公式为：  
![](/img/qt-book/paint/Paint-Base-Bezier-NM.png)

看上去是不是很复杂，难以理解？谁都一样，开始时看到这么复杂的公式，都会头大，但是看完下面的一阶，二阶，三阶贝塞尔曲线的方程和生成动画后就明白了，原来大名鼎鼎的贝塞尔曲线也不难嘛。

`一阶贝塞尔曲线` 就是线段，没有控制点，其参数方程为  
![](/img/qt-book/paint/Paint-Base-Bezier-1M.png)  
下图是生成一阶贝塞尔曲线的动画：  
![](/img/qt-book/paint/Paint-Base-Bezier-1M.gif)

`二阶贝塞尔曲线` 只有一个控制点，为 P1，其参数方程为  
![](/img/qt-book/paint/Paint-Base-Bezier-2M.png)  
下图是生成二阶贝塞尔曲线的动画：  
![](/img/qt-book/paint/Paint-Base-Bezier-2M.gif)

把其中的任意一帧拿出来分析，可以看到 MP0/MP1，NP1/Np2，KM/KN 都为 t/(1-t)，是不是一下又明白了很多，其他阶的贝塞尔曲线也是这样的。  
![](/img/qt-book/paint/Paint-Base-Bezier-Theory.png)

`三阶贝塞尔曲线` 有两个控制点，为 P1, P2，其参数方程为  
![](/img/qt-book/paint/Paint-Base-Bezier-3M.png)  
下图是生成三阶贝塞尔曲线的动画：  
![](/img/qt-book/paint/Paint-Base-Bezier-3M.gif)

贝塞尔曲线的更多介绍和动画请参考 <http://bbs.csdn.net/topics/390358020>。

绘制二阶贝塞尔曲线使用 `quadTo()`，第一个参数是控制点，第二个参数是曲线的终点

```cpp
void QPainterPath::quadTo(const QPointF &c, const QPointF &endPoint)
```

绘制三阶贝塞尔曲线使用 `cubicTo()`，第一个和第二个参数是控制点，第三个参数是曲线的终点

```cpp
void QPainterPath::cubicTo(const QPointF &c1, const QPointF &c2, const QPointF &endPoint)
```

或许你会问：为什么只看到了控制点和终点，没有看到起点？这是因为 QPainterPath 默认的起点在 (0, 0)，可以使用 moveTo() 改变起点，前一条线的终点就是下一条线的起点，结束亦是开始，人生亦是如此，生活处处皆道理，留心处处是学问，一花一世界，一叶一菩提，编程亦能悟道。

这里演示一个小程序，QPainterPath 添加了三条贝塞尔曲线，每条曲线有两个控制点，如图显示的 0 到 5 个共 6 个控制点，拖动控制点就会改变它的坐标，然后生成新的贝塞尔曲线并显示出来，实时的看到变化的结果。通过拖动控制点，可以生成各种不同的平滑曲线，这就是贝塞尔曲线的魅力所在。

![](/img/qt-book/paint/Paint-Base-Bezier-Demo.png)

```cpp
// 文件名：BezierCurveWidget.h
#ifndef BEZIERCURVEWIDGET_H
#define BEZIERCURVEWIDGET_H

#include <QWidget>
#include <QPointF>
#include <QList>
#include <QPainterPath>

class BezierCurveWidget : public QWidget {
    Q_OBJECT

public:
    explicit BezierCurveWidget(QWidget *parent = 0);
    ~BezierCurveWidget();

protected:
    void mousePressEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mouseMoveEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void paintEvent(QPaintEvent *event) Q_DECL_OVERRIDE;

    /**
     * 使用 breakPoints 和 controlPoints 创建贝塞尔曲线
     * 当控制点的坐标变化后都重新生成一次贝塞尔曲线
     */
    QPainterPath createBezierCurve();

    /**
     * 创建显示控制点的圆的 bounding rectangle
     * @param index 控制点在 controlPoints 中的下标
     */
    QRect createControlPointBundingRect(int index);

    /**
     * 为了设置坐标方便，从 (0, 0) 开始设置而不是实际绘制的坐标，对绘制贝塞尔曲线和控制点的
     * 坐标系做了偏移，在计算的时候，坐标也要是相对于新坐标系的坐标才行，不能是原始的坐标，
     * 所以要对其也好做相应的偏移
     * @param point - 例如鼠标按下时在 widget 上的原始坐标
     */
    QPointF translatedPoint(const QPointF &point) const;

private:
    QPainterPath bezierCurve;       // 贝塞尔曲线
    QList<QPointF *> breakPoints;   // 贝塞尔曲线端点的坐标
    QList<QPointF *> controlPoints; // 贝塞尔曲线控制点的坐标
    int pressedControlPointIndex;   // 鼠标按住的控制点在 controlPoints 里的下标
    int controlPointRadius;         // 显示控制点的圆的半径

    int translatedX; // 坐标系 X 轴的偏移量
    int translatedY; // 坐标系 Y 轴的偏移量
    int flags;       // 文本显示的参数
};

#endif // BEZIERCURVEWIDGET_H
```
<br>

```cpp
// 文件名：BezierCurveWidget.cpp
#include "BezierCurveWidget.h"
#include <QPainter>
#include <QMouseEvent>

BezierCurveWidget::BezierCurveWidget(QWidget *parent) : QWidget(parent) {
    // 贝塞尔曲线的端点
    breakPoints.append(new QPointF(0, 0));
    breakPoints.append(new QPointF(100, 100));
    breakPoints.append(new QPointF(200, 0));
    breakPoints.append(new QPointF(300, 100));

    // 第一段贝塞尔曲线控制点
    controlPoints.append(new QPointF(50, 0));
    controlPoints.append(new QPointF(50, 100));

    // 第二段贝塞尔曲线控制点
    controlPoints.append(new QPointF(150, 100));
    controlPoints.append(new QPointF(150, 0));

    // 第三段贝塞尔曲线控制点
    controlPoints.append(new QPointF(250, 0));
    controlPoints.append(new QPointF(250, 100));

    // 坐标设置好后就可以生成贝塞尔曲线了
    bezierCurve = createBezierCurve();

    controlPointRadius = 8;
    translatedX = 50;
    translatedY = 50;
    flags = Qt::AlignHCenter | Qt::AlignVCenter; // 水平和垂直居中
}

BezierCurveWidget::~BezierCurveWidget() {
    qDeleteAll(breakPoints);
    qDeleteAll(controlPoints);
}

void BezierCurveWidget::mousePressEvent(QMouseEvent *event) {
    pressedControlPointIndex = -1;

    // 绘制贝塞尔曲线和控制点的坐标系做了偏移，鼠标按下的坐标也要相应的偏移
    QPointF p = translatedPoint(event->pos());

    // 鼠标按下时，选择被按住的控制点
    for (int i = 0; i < controlPoints.size(); ++i) {
        QPainterPath path;
        path.addEllipse(*controlPoints.at(i), controlPointRadius, controlPointRadius);

        if (path.contains(p)) {
            pressedControlPointIndex = i;
            break;
        }
    }
}

void BezierCurveWidget::mouseMoveEvent(QMouseEvent *event) {
    // 移动选中的控制点
    if (pressedControlPointIndex != -1) {
        QPointF p = translatedPoint(event->pos());
        controlPoints.at(pressedControlPointIndex)->setX(p.x());
        controlPoints.at(pressedControlPointIndex)->setY(p.y());
        bezierCurve = createBezierCurve(); // 坐标发生变化后重新生成贝塞尔曲线
        update(); // 刷新界面
    }
}

QPainterPath BezierCurveWidget::createBezierCurve() {
    QPainterPath curve;
    curve.moveTo(*breakPoints.at(0));
    curve.cubicTo(*controlPoints[0], *controlPoints[1], *breakPoints[1]);
    curve.cubicTo(*controlPoints[2], *controlPoints[3], *breakPoints[2]);
    curve.cubicTo(*controlPoints[4], *controlPoints[5], *breakPoints[3]);

    return curve;
}

QRect BezierCurveWidget::createControlPointBundingRect(int index) {
    int x = controlPoints.at(index)->x() - controlPointRadius;
    int y = controlPoints.at(index)->y() - controlPointRadius;

    return QRect(x, y, controlPointRadius * 2, controlPointRadius * 2);
}

QPointF BezierCurveWidget::translatedPoint(const QPointF &point) const {
    return point - QPointF(translatedX, translatedY);
}

void BezierCurveWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);
    painter.translate(translatedX, translatedY);

    // 绘制贝塞尔曲线
    painter.drawPath(bezierCurve);

    // 绘制控制点和控制点的序号
    painter.setBrush(Qt::lightGray);
    for (int i = 0; i < controlPoints.size(); ++i) {
        QRect rect = createControlPointBundingRect(i);
        painter.drawEllipse(rect);
        painter.drawText(rect, flags, QString("%1").arg(i));
    }
}
```

```cpp
// 文件名：main.cpp
#include "BezierCurveWidget.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    BezierCurveWidget w;
    w.show();
    return a.exec();
}
```

这个程序不能像拖动控制点那样拖动曲线的端点改变贝塞尔曲线，这就作为留给大家的作业吧。