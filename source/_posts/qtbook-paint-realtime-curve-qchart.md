---
title: 使用 QChart 显示实时动态曲线
date: 2016-12-13 16:39:44
tags: QtBook
---

在 [实时动态曲线](/qtbook-paint-realtime-curve/) 一节介绍了使用算法实现实时动态曲线，Qt 5.7 后提供了 charts 模块，使用 QSplineSeries 就能很轻松的实现平滑曲线了，而且效果很好，但是需要注意一点的是，免费版的 Qt 中 charts 模块是 GPL 协议的。

效果如下，随着时间变化，曲线会从右向左移动

![](/img/qtbook/paint/Paint-RealTime-Curve-QChart.png)

<!--more-->
如果不会使用 QChart，可以先参考 [使用 QChart 创建平滑曲线](/qtbook-paint-smooth-curve-qchart) 后在看下面的实现代码。

```cpp
#ifndef REALTIMECURVEQCHARTWIDGET_H
#define REALTIMECURVEQCHARTWIDGET_H

#include <QWidget>
#include <QList>

#include <QSplineSeries>
#include <QScatterSeries>
#include <QChart>
#include <QChartView>

using namespace QtCharts;

class RealTimeCurveQChartWidget : public QWidget {
    Q_OBJECT

public:
    explicit RealTimeCurveQChartWidget(QWidget *parent = 0);
    ~RealTimeCurveQChartWidget();

protected:
    void timerEvent(QTimerEvent *event) Q_DECL_OVERRIDE;

private:
    /**
     * 接收到数据源发送来的数据，数据源可以下位机，采集卡，传感器等。
     */
    void dataReceived(int value);

    int timerId;
    int maxSize;  // data 最多存储 maxSize 个元素
    int maxValue; // 业务数据的最大值
    QList<double> data; // 存储业务数据的 list

    QChart *chart;
    QChartView *chartView;
    QSplineSeries *splineSeries;
    QScatterSeries *scatterSeries;
};

#endif // REALTIMECURVEQCHARTWIDGET_H
```

```cpp
#include "RealTimeCurveQChartWidget.h"
#include <QDateTime>
#include <QHBoxLayout>

RealTimeCurveQChartWidget::RealTimeCurveQChartWidget(QWidget *parent) : QWidget(parent) {
    maxSize = 31;   // 只存储最新的 31 个数据
    maxValue = 100; // 数据的最大值为 100，因为我们生成的随机数为 [0, 100]
    timerId = startTimer(200);
    qsrand(QDateTime::currentDateTime().toTime_t());

    splineSeries = new QSplineSeries();
    scatterSeries = new QScatterSeries();
    scatterSeries->setMarkerSize(8);

    // 预先分配坐标，这样在 dataReceived 中直接替换坐标了
    for (int i = 0; i < maxSize; ++i) {
        splineSeries->append(i * 10, -10);
        scatterSeries->append(i * 10, -10);
    }

    chart = new QChart();
    chart->addSeries(splineSeries);
    chart->addSeries(scatterSeries);
    chart->legend()->hide();
    chart->setTitle("实时动态曲线");
    chart->createDefaultAxes();
    chart->axisY()->setRange(0, maxValue);

    chartView = new QChartView(chart);
    chartView->setRenderHint(QPainter::Antialiasing);

    QHBoxLayout *layout = new QHBoxLayout();
    layout->setContentsMargins(0, 0, 0, 0);
    layout->addWidget(chartView);
    setLayout(layout);
}

RealTimeCurveQChartWidget::~RealTimeCurveQChartWidget() {
}

void RealTimeCurveQChartWidget::timerEvent(QTimerEvent *event) {
    if (event->timerId() == timerId) {
        // 模拟不停的接收到新数据
        int newData = qrand() % (maxValue + 1);
        dataReceived(newData);
    }
}

void RealTimeCurveQChartWidget::dataReceived(int value) {
    data << value;

    // 数据个数超过了指定值，则删除最先接收到的数据，实现曲线向前移动
    while (data.size() > maxSize) {
        data.removeFirst();
    }

    if (isVisible()) {
        // 界面被隐藏后就没有必要绘制数据的曲线了
        // 替换曲线中现有数据
        int delta = maxSize - data.size();
        for (int i = 0; i < data.size(); ++i) {
            splineSeries->replace(delta+i, splineSeries->at(delta+i).x(), data.at(i));
            scatterSeries->replace(delta+i, scatterSeries->at(delta+i).x(), data.at(i));
        }
    }
}
```

可能需要注意的可能就是 x 坐标的生成，因为数据是定时收到的，而且就只是一个数值，没有坐标信息，所以我们把其作为 y 坐标，x 坐标则为每隔 10 个像素就生成一个 x 坐标，这样在得到新的数据后，把数据一个挨一个的重新赋值给 series 就可以了，当然 x 坐标变化也是 0, 10, 20, ..., 100, ...，这里我们使用 replace() 函数来处理。

```cpp
#include "RealTimeCurveQChartWidget.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    RealTimeCurveQChartWidget w;
    w.show();

    return a.exec();
}
```

