---
title: Qt 实现 Steps 路径样式
date: 2017-07-30 19:35:53
tags: Qt
---

如下图使用多个 `步骤` 表示一个过程：

![](/img/qt/steps-1.png)

这样的组件 Qt 没有提供，需要我们自己实现，可以用下面几种方式实现:

* **使用 QPainter 绘图**：计算每一个步骤的图形(可以使用 QPainterPath)和位置，然后在 QPainterPath 上填充背景和文字

* **QPushButton + QSS Border-Image + 绝对坐标定位**：因为 QPushButton 之间有重叠，而不是一个紧挨着一个的排列，所以需要计算每个步骤的坐标进行定位，使用 PS 设计步骤在不同状态时的背景图，需要 6 张图片：

  * 当前步骤：第一个位置的图、中间的图、最后一个位置的图

  * 非当前步骤：第一个位置的图、中间的图、最后一个位置的图

  * 每个图是步骤的完整背景图，例如

      ![](/img/qt/steps-3.png)![](/img/qt/steps-4.png)

    > 优点：直观
    >
    > 缺点：需要手动计算坐标

* **QPushButton + QSS Border-Image + QHBoxLayout**：使用 Layout 把 QPushButton 一个紧挨着一个的排列，使用 PS 设计步骤在不同状态时的背景图，需要 5 张图片：

  * 当前步骤：当前步骤前一个步骤的图、最后一个位置的图

  * 非当前步骤：第一个位置的图、中间的图、最后一个位置的图

  * 每个图都不是步骤的完整背景图，例如

     ![](/img/qt/steps-6.png) ![](/img/qt/steps-5.png)

      > 优点：能够使用 Layout 进行布局，不需要手动计算坐标
      >
      > 缺点：不够直观，不过，在步骤之间加一点空隙，估计大家都明白怎么做了：每一个步骤的背景都有一份在它的前一个步骤上：
      >
      > ![](/img/qt/steps-7.png)


<!--more-->

下面介绍 **QPushButton + QSS Border-Image + QHBoxLayout** 的实现，其他两种方式感兴趣的话就自己思考一下怎么实现:

* 关键是类 StepWidget、style.qss 以及背景图
* 类 UiUtil 用于加载 style.qss，下载最后的工程文件进行查看
* main.cpp 用于启动程序

## StepWidget

```cpp
// 文件名: StepWidget.h

#ifndef STEPWIDGET_H
#define STEPWIDGET_H

#include <QWidget>
#include <QPushButton>
#include <QList>

class StepWidget : public QWidget {
    Q_OBJECT

public:
    StepWidget(QWidget *parent = 0);
    ~StepWidget();

protected slots:
    // 步骤按钮的槽，把被点击的按钮切换为当前步骤的按钮，然后更新样式
    void updateStepButtonsStyle();

private:
    // 把所有的按钮都放到 list 里，这样就很容易找到第一个，最后一个，当前的前一个等
    QList<QPushButton*> stepButtons;
};

#endif // STEPWIDGET_H
```

```cpp
// 文件名: StepWidget.cpp

#include "StepWidget.h"
#include "UiUtil.h"
#include <QPushButton>
#include <QHBoxLayout>
#include <QShortcut>
#include <QKeySequence>
#include <QDebug>

StepWidget::StepWidget(QWidget *parent) : QWidget(parent) {
    QHBoxLayout *layout = new QHBoxLayout();
    QStringList steps = QStringList() << "提交订单" << "付款成功" << "商品出库" << "------ 快递中-等待收货 ------" << "完成";

    // 创建 steps.size() 个按钮，每个按钮表示一个状态
    foreach (const QString &step, steps) {
        QPushButton *button = new QPushButton(step);
        button->setFlat(true);
        button->setProperty("class", "StepButton");
        button->setProperty("status", "middle");
        stepButtons.append(button);
        layout->addWidget(button);

        connect(button, SIGNAL(clicked()), this, SLOT(updateStepButtonsStyle()));
    }
    stepButtons.last()->setProperty("status", "last");
    stepButtons.at(2)->click();

    layout->addStretch();
    layout->setSpacing(0);
    setLayout(layout);
}

StepWidget::~StepWidget() {
}

void StepWidget::updateStepButtonsStyle() {
    QPushButton *clickedButton = qobject_cast<QPushButton*>(sender()); // 被点击的按钮
    int clickedIndex = stepButtons.indexOf(clickedButton);
    QPushButton *prevButton = stepButtons.value(clickedIndex - 1); // 被点击的按钮的前一个按钮
    bool isLast = (clickedIndex == stepButtons.size() - 1); // 是否点击最后一个按钮

    // 恢复所有按钮的 status 为初始状态
    foreach (QPushButton *b, stepButtons) {
        b->setProperty("status", "middle");
    }
    stepButtons.last()->setProperty("status", "last");

    // 设置被点击的按钮的 status 属性为 active，如果是最后一个按钮则为 active-last
    clickedButton->setProperty("status", isLast ? "active-last" : "active-middle");

    // 设置被点击的按钮的前一个按钮的 status 为 active-prev
    prevButton && prevButton->setProperty("status", "active-prev");

    // 属性变化后，使属性选择器的 QSS 生效
    this->setStyleSheet("/**/");
}
```

给按钮设置 class 属性为 StepButton，是为了它们的 QSS 不影响正常 QPushButton 的样式。

按钮有 5 个状态: middle, last, active-middle, active-last, active-prev

* 初始时第一个到最后一个的状态是 middle，最后一个的状态是 last
* 当点击一个按钮后，它变为当前按钮，它的状态变为 active-middle，如果它是最后一个按钮，则为 active-last
* 当前按钮的前一个按钮的状态是 active-prev
* 按钮的属性变化后对应的 QSS 不会自动生效，需要调用 `setStyleSheet("/**/")` 才行

## style.qss

```css
.StepButton {
    min-width: 80px;
    min-height: 40px;
    max-height: 40px;
    padding-left: 10px;
    padding-right: -10px;
    border-width: 0px 22px 0px 0px;
}

.StepButton[status="middle"] {
    /* repeat: 水平平铺，stretch: 垂直拉伸 */
    border-image: url(:/img/normal.png) 0 22 0 0 repeat stretch;
}

.StepButton[status="last"] {
    border-image: url(:/img/last.png) 0 22 0 0 repeat stretch;
}

.StepButton[status="active-middle"] {
    border-image: url(:/img/active.png) 0 22 0 0 repeat stretch;
}

.StepButton[status="active-last"] {
    border-image: url(:/img/last-active.png) 0 22 0 0 repeat stretch;
}

.StepButton[status="active-prev"] {
    border-image: url(:/img/active-pre.png) 0 22 0 0 repeat stretch;
}

QWidget {
    background: gray;
}
```

上面 QSS 的关键是 Border-Image 的应用，相关教程请参考 [Border Image](/qtbook-qss-border-image/)

## 工程源码

[Qt-Steps.7z](/download/Qt-Steps.7z)