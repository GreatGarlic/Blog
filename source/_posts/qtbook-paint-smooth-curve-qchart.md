---
title: 使用 QChart 创建平滑曲线
date: 2016-12-13 11:27:12
tags: QtBook
---

在 [绘制平滑曲线](/qtbook-paint-smooth-curve) 一节介绍了使用算法实现平滑曲线，Qt 5.7 后提供了 `charts` 模块，使用 `QSplineSeries` 就能很轻松的实现平滑曲线了，而且效果很好，但是需要注意一点的是，免费版的 Qt 中 charts 模块是 GPL 协议的。<!--more-->

实现如图效果的平滑曲线，只需要简单的几步就可以做到，具体请参考下面的代码

![](/img/qtbook/paint/Paint-SmoothCurve-QChart.png)

```cpp
#include <QApplication>
#include <QWidget>
#include <QChartView>
#include <QSplineSeries>
#include <QScatterSeries>
#include <QChart>
#include <QHBoxLayout>

using namespace QtCharts;

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    // 创建平滑曲线上点的序列
    QSplineSeries *splineSeries = new QSplineSeries();
    splineSeries->setName("spline");

    splineSeries->append(0, 6);
    splineSeries->append(2, 4);
    splineSeries->append(3, 8);
    splineSeries->append(7, 4);
    splineSeries->append(10, 5);
    *splineSeries << QPointF(11, 1) << QPointF(13, 3) << QPointF(17, 6) << QPointF(18, 3) << QPointF(20, 2);

    // 创建散列点的序列
    QScatterSeries *scatterSeries = new QScatterSeries();
    scatterSeries->setMarkerSize(8);
    scatterSeries->append(0, 6);
    scatterSeries->append(2, 4);
    scatterSeries->append(3, 8);
    scatterSeries->append(7, 4);
    scatterSeries->append(10, 5);
    *scatterSeries << QPointF(11, 1) << QPointF(13, 3) << QPointF(17, 6) << QPointF(18, 3) << QPointF(20, 2);

    // 使用点的序列创建图标
    QChart *chart = new QChart();
    chart->legend()->hide();
    chart->addSeries(splineSeries);
    chart->addSeries(scatterSeries);
    chart->setTitle("平滑曲线");
    chart->createDefaultAxes();
    chart->axisY()->setRange(0, 10);

    // 显示图标的 view
    QChartView *chartView = new QChartView(chart);
    chartView->setRenderHint(QPainter::Antialiasing);

    // Widget 相关
    QHBoxLayout *layout = new QHBoxLayout();
    layout->setContentsMargins(0, 0, 0, 0);
    layout->addWidget(chartView);

    QWidget w;
    w.setWindowTitle("QChart 实现的平滑曲线");
    w.resize(600, 300);
    w.setLayout(layout);
    w.show();

    return a.exec();
}
```

为了显示平滑曲线，分为以下几步

1. 创建 QSplineSeries 对象 splineSeries，并且把曲线上点的坐标添加到 splineSeries，QSplineSeries 会自动的计算曲线的控制点，这些控制点是绘制平滑曲线的关键，就像前面的文章中我们使用算法计算的控制点那样，只不过我们当时提供的算法不是很好

   ```cpp
   QSplineSeries *splineSeries = new QSplineSeries();
   splineSeries->append(0, 6);
   ```

2. 曲线的数据准备好后，需要放在一个 QChart 里显示，一个 QChart 可以同时显示几个 Series

   ```cpp
   QChart *chart = new QChart();
   chart->addSeries(splineSeries);
   ```

3. 最后，用 QChart 来创建一个 QChartView，最终显示给用户，QChartView 也可以作为另一个 Widget 的子 Widget

   ```cpp
   QChartView *chartView = new QChartView(chart);
   ```


更多信息请参考 Qt 自带的例子，搜索 chart example

![](/img/qtbook/paint/Paint-SmoothCurve-QChart-Example.png)