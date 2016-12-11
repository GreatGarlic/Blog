---
title: 绘制平滑曲线
date: 2016-12-11 10:33:47
tags: QtBook
---
得到曲线上的点，画出曲线，这是一个很常见的需求。画曲线嘛，当然难不住我们，用 QPainter::drawLine() 把曲线上的点连起来不就好了？So easy，轻轻松松搞定，开开心心的交任务去了。

![](/img/qtbook/paint/Paint-SmoothCurve-Non.png)

正在聚精会神炒股的老板一瞅，气不打一处来：“你这画的是什么鬼，这个线直来直去的，太不专业了”，抬头指着屏幕上的炒股软件，瞅着迷离的眼神：“看看人家的这个曲线，就像少女的皮肤般那么的柔顺、平滑”，口气马上一百八十度大转弯：“在看看你的，像八十岁老头的那样全是褶皱！” 擦完脸上的口水，赶快想办法去吧。<!--more-->

创建平滑曲线，有很多种方式，可以使用现成的库，例如 QWT，QCustomPlot 等，也可以研究平滑曲线的理论，实现平滑曲线的插值函数。这里我们提供一个实现，利用 QPainterPath，根据曲线上的点创建平滑曲线，使用分三步：

```cpp
// [1] 把曲线上的点放到 QList 里
QList<QPointF> knots;
knots << QPointF(x1, y1);
...
knots << QPointF(xn, yn);

// [2] 创建平滑曲线
QPainterPath smoothCurve = SmoothCurveGenerator::generateSmoothCurve(knots);

// [3] 绘制曲线
painter.drawPath(smoothCurve);
```

最后画出来的曲线效果如下，这次应该不会被喷了吧：

![](/img/qtbook/paint/Paint-SmoothCurve.png)

Qt 中可以使用 `QPainterPath::cubicTo()` 函数实现绘制平滑曲线，绘制平滑曲线的关键是控制点的计算，sp 为线段的起始点，ep 为线段的终点，c1，c2 为贝塞尔曲线的控制点，其坐标计算如下

![](/img/qtbook/paint/Paint-SmoothCurve-Theory.png)

类 SmoothCurveGenerator 使用上面的原理生成平滑曲线

```cpp
#ifndef SMOOTHCURVEGENERATOR_H
#define SMOOTHCURVEGENERATOR_H

#include <QList>
#include <QPointF>
#include <QPainterPath>

class SmoothCurveGenerator {
public:
    /**
     * 传入曲线上的点的 list，创建平滑曲线
     *
     * @param points - 曲线上的点
     * @return - 返回使用给定的点创建的 QPainterPath 表示的平滑曲线
     */
    static QPainterPath generateSmoothCurve(const QList<QPointF> &points);
};

#endif // SMOOTHCURVEGENERATOR_H
```

```cpp
#include "SmoothCurveGenerator.h"

QPainterPath SmoothCurveGenerator::generateSmoothCurve(const QList<QPointF> &points) {
    if (points.size() == 0) {
        return QPainterPath();
    }

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

    return path;
}
```

接下来就介绍一下 SmoothCurveGenerator 的使用案例，就是开头被喷的那个曲线图的平滑实现。

在 SmoothCurveWidget.ui 上如图放两个 QCheckBox，命名为 showKnotsCheckBox 和 showSmoothCurveCheckBox，还有一个 QPushButton，用于点击重新生成曲线：

![](/img/qtbook/paint/Paint-SmoothCurve-Ui.png)

下面是 SmoothCurveWidget.h, SmoothCurveWidget.cpp 和 main.cpp

```cpp
#ifndef SMOOTHCURVEWIDGET_H
#define SMOOTHCURVEWIDGET_H

#include <QWidget>
#include <QList>
#include <QPointF>
#include <QPainterPath>

namespace Ui {
class SmoothCurveWidget;
}

class SmoothCurveWidget : public QWidget {
    Q_OBJECT

public:
    explicit SmoothCurveWidget(QWidget *parent = 0);
    ~SmoothCurveWidget();

protected:
    void paintEvent(QPaintEvent *event) Q_DECL_OVERRIDE;

private slots:
    void generateCurves(); // 生成平滑曲线和非平滑曲线

private:
    Ui::SmoothCurveWidget *ui;
    QList<QPointF> knots;        // 曲线上的点
    QPainterPath smoothCurve;    // 平滑曲线
    QPainterPath nonSmoothCurve; // 直接连接点的非平滑曲线
};

#endif // SMOOTHCURVEWIDGET_H
```

```cpp
#include "SmoothCurveWidget.h"
#include "ui_SmoothCurveWidget.h"
#include "SmoothCurveGenerator.h"

#include <QPainter>
#include <QtGlobal>
#include <QDateTime>

SmoothCurveWidget::SmoothCurveWidget(QWidget *parent) :
    QWidget(parent), ui(new Ui::SmoothCurveWidget) {
    ui->setupUi(this);

    connect(ui->generateCurveButton, SIGNAL(clicked(bool)), this, SLOT(generateCurves()));
    connect(ui->showKnotsCheckBox, SIGNAL(clicked(bool)), this, SLOT(update()));
    connect(ui->showSmoothCurveCheckBox, SIGNAL(clicked(bool)), this, SLOT(update()));

    ui->generateCurveButton->click();
}

SmoothCurveWidget::~SmoothCurveWidget() {
    delete ui;
}

void SmoothCurveWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);
    painter.translate(width() / 2 , height() / 2);
    painter.scale(1, -1);

    // 画坐标轴
    painter.setPen(QColor(180, 180, 180));
    painter.drawLine(-500, 0, 500, 0);
    painter.drawLine(0, 500, 0, -500);

    // showSmoothCurveCheckBox 被选中时显示平滑曲线，否则显示非平滑曲线
    painter.setPen(QPen(QColor(80, 80, 80), 2));
    painter.drawPath(ui->showSmoothCurveCheckBox->isChecked() ? smoothCurve : nonSmoothCurve);

    // 如果曲线上的点可见，则显示出来
    if (ui->showKnotsCheckBox->isChecked()) {
        painter.setPen(Qt::black);
        painter.setBrush(Qt::gray);
        foreach(QPointF p, knots) {
            painter.drawEllipse(p, 3, 3);
        }
    }
}

void SmoothCurveWidget::generateCurves() {
    qsrand(QDateTime::currentDateTime().toMSecsSinceEpoch());

    // 随机生成曲线上的点: 横坐标为 [-200, 200]，纵坐标为 [-100, 100]
    int x = -200;
    knots.clear();
    while (x < 200) {
        knots << QPointF(x, qrand() % 200 - 100);
        x += qMin(qrand() % 30 + 5, 200);
    }

    // 根据曲线上的点创建平滑曲线
    smoothCurve = SmoothCurveGenerator::generateSmoothCurve(knots);

    // 连接点创建非平滑曲线曲线
    nonSmoothCurve = QPainterPath();
    nonSmoothCurve.moveTo(knots[0]);
    for (int i = 1; i < knots.size(); ++i) {
        nonSmoothCurve.lineTo(knots[i]);
    }

    update();
}
```

```cpp
#include "SmoothCurveWidget.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    SmoothCurveWidget w;
    w.show();

    return a.exec();
}
```

为了让程序更具有普谝性，曲线上的点采用随机生成，所以每次生成的曲线是不一样的，选中 "Smooth Curve" 时显示为平滑曲线，没有选中则显示为非平滑的曲线。选中 "Show knots" 显示曲线上的点，反之则不显示。