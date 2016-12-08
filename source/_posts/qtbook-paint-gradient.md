---
title: 渐变及原理
date: 2016-12-08 23:17:42
tags: Qt-Book
---

渐变有三种：QLinearGradient, QConicalGradient and QRadialGradient，接下来将介绍它们的使用和实现原理。<!--more-->

## QLinearGradient
QLinearGradient 是线性渐变，也就是颜色的各个分量（red, green, blue）在两点之间的变化是线性的，需要设置渐变的起始和结束坐标、颜色，超出渐变范围的填充方式，它并不能单独的使用，而是要和 QBrush 一起使用实现填充效果，主要有以下一些函数：

```cpp
// 创建 QLinearGradient，同时设置起始和结束坐标
QLinearGradient(const QPointF &start, const QPointF &finalStop)
QLinearGradient(qreal x1, qreal y1, qreal x2, qreal y2)

// 设置渐变的颜色，position 的取值范围是 [0.0, 1.0]
setColorAt(qreal position, const QColor &color)

// 超出渐变范围后的填充方式，默认使用 PadSpread: 
//     QGradient::PadSpread
//     QGradient::RepeatSpread
//     QGradient::ReflectSpread
void setSpread(Spread method)

// 使用渐变创建画刷
QBrush(const QGradient &gradient)
```

下图来自 QLinearGradient 的帮助文档，两个灰色的点表示渐变的起始和结束位置，从黄色渐变到有点发灰的黄色，同时展示了超出渐变范围时的三种填充方式：

![](/img/qt-book/paint/Paint-Base-LinearGradient.png)

为了介绍 QLinearGradient 的使用，下面的程序使用线性渐变，在垂直方向从红色渐变到蓝色，填充矩形 QRect(20, 20, 200, 200)：

![](/img/qt-book/paint/Paint-Base-LinearGradient-Demo-1.png)

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);
    QRect rect(20, 20, 200, 200);

    // 渐变开始的坐标为 (20, 20), 结束的坐标为 (20, 220)
    QLinearGradient gradient(rect.x(), rect.y(),
                             rect.x(), rect.y() + rect.height());
    gradient.setColorAt(0.0, Qt::red);
    gradient.setColorAt(1.0, Qt::blue);
    // 超出渐变范围后的填充方式
    gradient.setSpread(QGradient::ReflectSpread);

    painter.setPen(Qt::NoPen);
    painter.setBrush(gradient); // QBrush(const QGradient &gradient)
    painter.drawRect(rect);
}
```

如果不用 QLinearGradient，怎么实现上面的渐变效果呢？也既是线性渐变的原理是什么呢？  
以求线段上任意点的坐标为例，如图，已知线段的两端点 A(x1, y1)，B(x2, y2)，求线段上任意一点 M 的坐标 (x, y)，则

![](/img/qt-book/paint/Paint-Base-LinearGradient-Coordinate.png)

```
根据两点的距离公式可以求出线段的长度 |AB|（用 || 表示线段的长度）
t = |AM| / |AB|;
因为 |AM| >= 0 且 |AM| <= |AB|，所以 t 的值为 [0.0, 1.0]，用 length 表示 |AB|，则
x = x1 + t * length
y = y1 + t * length

t 为 0.0 时 M 和 A 重合，t 为 1.0 时 M 和 B 重合。
因为 t 的值为 0 到 1 之间，所以可以用循环求出 AB 上任意点的坐标 

for (float t = 0.0; rate <= 1.0; t += 0.1) {
    x = x1 + t * length;
    y = y1 + t * length;
}

其实这就是线段的参数方程。
```

上面可以理解为坐标的渐变，变化的是坐标的 x, y 分量，颜色的渐变理论上也是一样的，只不过要变化的是颜色的 R, G, B 三个分量。如果同时已知点 A，B 的坐标和颜色 (r1, g1, b1), (r2, g2, b2)，那么 M 点的坐标和颜色为：

```cpp

for (float t = 0.0; rate <= 1.0; t += 0.1) {
    x = x1 + t * length;
    y = y1 + t * length;
    
    r = r1 + t * (r2-r1);
    g = g1 + t * (g2-g1);
    b = b1 + t * (b2-b1);
}
```

也既是说，如果知道某个点对应的 t，那么就能计算出此点的颜色。如下图，要在矩形内沿着 AB 进行渐变填充，已知点 A，B 的坐标和颜色，在矩形内任意一点 N 的坐标也是已知的（循环遍历矩形内所有的点），那么就可以求出点 N 在 AB 上的投影 M（MN 垂直于 AB），t=|AM|/|AB|，使用上面的方法求出点 M 的颜色，点 M 的颜色就是点 N 的颜色。

![](/img/qt-book/paint/Paint-Base-LinearGradient-Color.png)

对于下图垂直方向的渐变来说，点 A(x1, y1) 为矩形的左上角，点 B(x2, y2) 为矩形的坐下角，矩形内任意一点 N(x, y) 在 AB 上的投影 M 的坐标为 (x1, y)，所以 t = (y-y1)/(y2-y1)，知道了 t，那么就能计算出对应的颜色了。

![](/img/qt-book/paint/Paint-Base-LinearGradient-Demo-1.png)

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);

    // 渐变填充的矩形
    QRect rect(20, 20, 200, 200);

    // 渐变开始和结束的颜色
    QColor gradientStartColor(255, 0, 0);
    QColor gradientFinalColor(0, 0, 255);

    int r1, g1, b1, r2, g2, b2;
    gradientStartColor.getRgb(&r1, &g1, &b1);
    gradientFinalColor.getRgb(&r2, &g2, &b2);

    qreal y1 = rect.y();
    qreal y2 = rect.y() + rect.height();

    // 计算矩形中每一个点的颜色，然后用此颜色绘制这个点
    for (int x = rect.x(); x <= rect.x() + rect.width(); ++x) {
        for (int y = rect.y(); y <= rect.y() + rect.height(); ++y) {
            qreal t = (y-y1) / (y2-y1);
            t = qMax(0.0, qMin(t, 1.0));

            int r = r1 + t * (r2-r1);
            int g = g1 + t * (g2-g1);
            int b = b1 + t * (b2-b1);

            painter.setPen(QColor(r, g, b));
            painter.drawPoint(x, y);
        }
    }
}
```

运行程序，看看效果是不是和使用 QLinearGradient 的一样？

对于下图这样指定渐变的开始和结束位置，非垂直和水平方向渐变的实现，关键是求任意一点在另一条线上的投影，有很多方法和公式可以使用，这里我们使用 QTransform 进行移动，旋转求出 t，计算出对应的颜色，由于 QTransform 的知识比较复杂，这里就不作深入介绍，有兴趣的可以自行查看相关文档。

![](/img/qt-book/paint/Paint-Base-LinearGradient-Demo-2.png)

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);

    // 渐变填充的矩形
    QRect rect(20, 20, 200, 200);

    // 渐变开始和结束的颜色、位置
    QColor gradientStartColor(255, 0, 0);
    QColor gradientFinalColor(0, 0, 255);
    QPoint gradientStartPoint(60, 60);
    QPoint gradientFinalPoint(180, 180);

    // 颜色分量
    int r1, g1, b1, r2, g2, b2;
    gradientStartColor.getRgb(&r1, &g1, &b1);
    gradientFinalColor.getRgb(&r2, &g2, &b2);

    qreal dx = gradientFinalPoint.x() - gradientStartPoint.x();
    qreal dy = gradientFinalPoint.y() - gradientStartPoint.y();
    qreal length = qSqrt(dx*dx + dy*dy); // 渐变开始和结束的线段的长度
    float radian = qAtan2(dy, dx); // 渐变方向和 X 轴的夹角

    // 先移动，后旋转，要先调用旋转的函数，然后在调用移动的函数，一定要注意这点，
    // 因为底层实现是 matrix 矩阵右乘点的坐标的列矩阵
    QTransform transform;
    transform.rotateRadians(-radian);
    transform.translate(-gradientStartPoint.x(), -gradientStartPoint.y());

    // 计算矩形中每一个点的颜色，然后用此颜色绘制这个点
    for (int x = rect.x(); x <= rect.x() + rect.width(); ++x) {
        for (int y = rect.y(); y <= rect.y() + rect.height(); ++y) {
            QPointF p = transform.map(QPointF(x ,y));
            qreal t = p.x() / length;
            t = qMax(0.0, qMin(t, 1.0));

            int r = r1 + t * (r2-r1);
            int g = g1 + t * (g2-g1);
            int b = b1 + t * (b2-b1);

            painter.setPen(QColor(r, g, b));
            painter.drawPoint(x, y);
        }
    }
}
```

t<0 或 t>1 时，即超出渐变范围后的填充方式是需要考虑的，我们这里的实现就是 PadSpread 的方式，怎么实现 RepeatSpread 和 ReflectSpread 的渐变呢？这个就作为大家的作业吧。

## QRadialGradient
QRadialGradient 名为 `径向渐变`，在圆的范围内进行渐变，有三个主要参数：圆心、半径、焦点：

```cpp
QRadialGradient(const QPointF &center, qreal radius, 
                const QPointF &focalPoint)
QRadialGradient(const QPointF & center, qreal radius)
```

圆心和半径确定颜色渐变的范围，焦点是渐变开始的点，渐变结束的点在圆周上。很多人都认为径向渐变是从圆心开始渐变的，其实不是这样的，只不过焦点和圆心默认是在同一个位置，所以看上去渐变好像是从圆心开始。

如图，我们故意设置圆心和焦点不在同一个位置，这样就能很明显的看到渐变的范围，开始和结束的位置，连接焦点和圆周的线上的点的颜色做线性渐变（是不是知道怎么实现 QRadialGradient 了？）。

![](/img/qt-book/paint/Paint-Base-RadialGradient.png)

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);
    painter.translate(width() / 2, height() / 2);

    qreal radius = 150;    // 半径
    QPointF center(0, 0);  // 圆心
    QPointF focus(80, 30); // 焦点

    // 径向渐变
    QRadialGradient gradient(center, radius, focus);
    gradient.setColorAt(0.0, Qt::red);
    gradient.setColorAt(1.0, Qt::blue);

    // 径向渐变填充圆
    painter.setPen(Qt::darkGray);
    painter.setBrush(gradient);
    painter.drawEllipse(center, radius, radius);

    // 绘制圆心和焦点
    painter.setBrush(Qt::gray);
    painter.drawEllipse(center, 4, 4);
    painter.drawEllipse(focus, 4, 4);
}
```

## QConicalGradient
QConicalGradient 名为 `角度渐变`，只需要指定渐变的中心和开始的角度：

```cpp
QConicalGradient(const QPointF &center, qreal angle)
QConicalGradient(qreal cx, qreal cy, qreal angle)
```

经过线性渐变和径向渐变的学习，相信现在大家都能很容易的推断得出角度渐变的原理，这里就不作解释，作为悬念留给大家吧。

![](/img/qt-book/paint/Paint-Base-ConicalGradient.png)

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);
    painter.translate(width() / 2, height() / 2);

    qreal startAngle = 45; // 渐变开始的角度
    QPointF center(0, 0);  // 渐变的中心

    QConicalGradient gradient(center, startAngle);
    gradient.setColorAt(0.0, Qt::red);
    gradient.setColorAt(0.33, Qt::green);
    gradient.setColorAt(0.66, Qt::blue);
    gradient.setColorAt(1.0, Qt::red);

    painter.setPen(Qt::darkGray);
    painter.setBrush(gradient);
    painter.drawEllipse(center, 150, 150);
}
```