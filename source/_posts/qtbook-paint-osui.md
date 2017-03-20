---
title: 襁褓中的系统界面
date: 2017-03-19 20:37:30
tags: QtBook
---

是否还记得在开始的时候我们说过：

* 给我一个 **QPainter**，我也能实现整个操作系统的图形界面
* 操作系统的界面本质也是**画(Hua)**出来的
* 图形界面的本质都是一样的，就是一张静态的画
* 点击按钮，看到按钮动了

这里我们就用 QPainter 绘图来模拟实现一个系统的界面原型，为了简单说明问题，只绘制了 Button 和 CheckBox，其他的控件同理。当然 Button 是能够点击的，点击 CheckBox 也能够切换选中状态，效果如下图:

![](/img/qtbook/paint/Paint-OSUi-1.png) ![](/img/qtbook/paint/Paint-OSUi-2.png)

<!--more-->

## Widget

Widget 是所有控件的所有控件的父类，实现控件共有的逻辑，Button 和 CheckBox 是具体的控件，不同控件的行为和样式都不一样，都需要在自己的类中实现。

```cpp
// 文件名: Widget.h
#ifndef WIDGET_H
#define WIDGET_H
#include <QRect>

class QPainter;

/**
 * 控件的基类 Widget，所有控件类都需要直接或者间接的继承类 Widget，它处理控件类的一些共有的逻辑。
 */
class Widget {
public:
    Widget(const QRect &boundingRect);

    // 绘制控件的函数，不同的控件提供不同的实现，绘制出不同的样子。
    // paint() 是纯虚函数，控件子类必须提供它的实现。
    virtual void paint(QPainter *painter) = 0;

    // 一下几个为鼠标事件
    virtual void mouseMove();     // 鼠标移动事件
    virtual void mouseEnter();    // 鼠标进入事件
    virtual void mouseLeave();    // 鼠标离开事件
    virtual void mousePressed();  // 鼠标按下事件
    virtual void mouseReleased(); // 鼠标按下后松开事件

    bool  hover;        // 鼠标移动到 widget 上时为 true，其他时候为 false
    bool  pressed;      // 鼠标在按住 widget 时为 true，其他时候为 false
    QRect boundingRect; // Widget 的范围: 左上角的坐标和宽、高
};

#endif // WIDGET_H
```

```cpp
// 文件名: Widget.cpp
#include "Widget.h"

Widget::Widget(const QRect &boundingRect)
    : hover(false), pressed(false), boundingRect(boundingRect) {
}

void Widget::mouseMove() {

}

void Widget::mouseEnter() {
    hover = true;
}

void Widget::mouseLeave() {
    hover = false;
}

void Widget::mousePressed() {
    pressed = true;
}

void Widget::mouseReleased() {
    pressed = false;
}
```

## Button

下面的代码只做出了鼠标移动到 Button 上，鼠标点击时的不同高亮效果，点击 Button 时没有发射 clicked() 事件。

如果需要实现点击时发射 clicked() 信号也容易，鼠标事件的时候把鼠标的坐标发送给 Button，在 mouseReleased() 事件发生时如果鼠标仍然在 Button 上，发射 clicked() 信号即可，和这个信号关联的槽函数就能被调用了，实现点击的事件处理。传递坐标的时候最好把 parent 的坐标映射为 child 的坐标，即 parent 的坐标减去 Button 在 parent 中左上角的坐标即可。

```cpp
// 文件名: Button.h
#ifndef BUTTON_H
#define BUTTON_H

#include "Widget.h"

#include <QRect>
#include <QColor>
#include <QPainter>

/**
 * 按钮类，可以定义按钮的背景色，鼠标放在它上面时的背景色以及鼠标按下时的背景色等。
 */
class Button : public Widget {
public:
    Button(const QString &text, const QRect &boundingRect = QRect(0, 0, 100, 25),
           const QColor &normalBackgroundColor = QColor(200, 200, 200),
           const QColor &hoverBackgroundColor = QColor(200, 0, 200),
           const QColor &pressedBackgroundColor = QColor(0, 200, 200));

    void paint(QPainter *painter) Q_DECL_OVERRIDE;

    QString text; // 按钮的文本
    QColor  normalBackgroundColor;  // 背景色
    QColor  hoverBackgroundColor;   // 鼠标放到按钮上的背景色
    QColor  pressedBackgroundColor; // 鼠标按下按钮的背景色
};

#endif // BUTTON_H
```

```cpp
// 文件名: Button.cpp
#include "Button.h"

Button::Button(const QString &text, const QRect &boundingRect,
               const QColor &normalBackgroundColor,
               const QColor &hoverBackgroundColor,
               const QColor &pressedBackgroundColor)
    : Widget(boundingRect), text(text),
      normalBackgroundColor(normalBackgroundColor),
      hoverBackgroundColor(hoverBackgroundColor),
      pressedBackgroundColor(pressedBackgroundColor) {
}

void Button::paint(QPainter *painter) {
    QColor backgroundColor; // 按钮的背景色

    if (pressed) {
        backgroundColor = pressedBackgroundColor;
    } else if (hover) {
        backgroundColor = hoverBackgroundColor;
    } else {
        backgroundColor = normalBackgroundColor;
    }

    // 先绘制按钮的背景，然后居中绘制按钮的文本
    painter->setBrush(QBrush(backgroundColor));
    painter->drawRoundedRect(boundingRect, 2, 2);
    painter->drawText(boundingRect, Qt::AlignCenter, text);
}
```

## CheckBox

```cpp
// 文件名: CheckBox.h
#ifndef CHECKBOX_H
#define CHECKBOX_H
#include "Widget.h"
#include <QString>
#include <QRect>

class QPainter;

/**
 * CheckBox 类，有选中和未被选中状态。
 */
class CheckBox : public Widget {
public:
    CheckBox(const QString &text, bool checked = true, const QRect &boundingRect = QRect(0, 0, 100, 25));
    void paint(QPainter *painter) Q_DECL_OVERRIDE;
    void mousePressed() Q_DECL_OVERRIDE;

    bool checked; // 选中时为 true，否则为 false
    QString text; // CheckBox 的文本
};

#endif // CHECKBOX_H
```

```cpp
// 文件名: CheckBox.cpp
#include "CheckBox.h"
#include <QPainter>

CheckBox::CheckBox(const QString &text, bool checked, const QRect &boundingRect)
    : Widget(boundingRect), checked(checked), text(text) {
}

void CheckBox::paint(QPainter *painter) {
    painter->translate(boundingRect.x(), boundingRect.y());

    int w = boundingRect.width();
    int h = boundingRect.height();

    // 绘制 Indicator 的边框
    painter->setPen(QPen(Qt::darkGray, 2));
    painter->drawRect(0, 0, h, h);

    // 选中时，Indicator 内部绘制一个勾
    if (checked) {
        painter->setPen(QPen(Qt::darkGray, 3, Qt::SolidLine, Qt::RoundCap));
        painter->drawLine(h*0.2, h*0.5,  h*0.4, h*0.75);
        painter->drawLine(h*0.4, h*0.75, h*0.9, h*0.3);
    }

    // 绘制 CheckBox 的文本
    painter->setPen(Qt::black);
    painter->drawText(h+10, 0, w-h-10, h, Qt::AlignLeft|Qt::AlignVCenter, text);
}

void CheckBox::mousePressed() {
    checked = !checked; // 鼠标按下时切换选中状态
    // 可以发射 statusChanged 信号
}
```

## OSUi

OSUi 就假设是操作系统的界面吧，各种控件都是放置在它的上面，当 OSUi 接收到鼠标事件后，会查找此时鼠标在哪个控件上，然后就把鼠标事件传递给对应的控件，然后控件对此鼠标事件作出自己特有的响应。

因为所有的控件都继承自 Widget，所以用一个 List 保存了所有控件的指针，paintEvent() 更新界面时调 Widget::paint() 把所有的控件都重新绘制一次，需要注意的是每个 Widget::paint() 调用的时候，都需要保存一下 QPainter 的状态，绘制完后恢复，避免不同的控件使用 QPainter 后影响到其他控件的绘制。

```cpp
// 文件名: OSUi.h
#ifndef OSUI_H
#define OSUI_H

#include <QWidget>
#include <QList>

class Widget;

/**
 * 操作系统的界面，在它的上面放置各种控件。
 */
class OSUi : public QWidget {
    Q_OBJECT

public:
    explicit OSUi(QWidget *parent = 0);
    ~OSUi();

protected:
    void paintEvent(QPaintEvent *event) Q_DECL_OVERRIDE;
    void mouseMoveEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mousePressEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mouseReleaseEvent(QMouseEvent *event) Q_DECL_OVERRIDE;

private:
    QList<Widget*> widgets; // 界面上所有的控件的集合
};

#endif // OSUI_H
```

```cpp
// 文件名: OSUi.cpp
#include "OSUi.h"
#include "Button.h"
#include "CheckBox.h"

#include <QRect>
#include <QPainter>
#include <QMouseEvent>

OSUi::OSUi(QWidget *parent) : QWidget(parent) {
    setMouseTracking(true);

    // 创建界面上的所有控件
    widgets << new Button("按钮一", QRect(20, 20, 100, 25))
            << new Button("按钮二", QRect(20, 70, 100, 25), QColor(200, 200, 200), QColor(200, 200, 0), QColor(0, 200, 0))
            << new CheckBox("生杀大权在你手里", true, QRect(150, 50, 150, 25));
}

OSUi::~OSUi() {
    qDeleteAll(widgets);
}

void OSUi::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing, true);

    // 每次更新都需要绘制所有的控件
    foreach (Widget *w, widgets) {
        painter.save();
        w->paint(&painter);
        painter.restore();
    }
}

void OSUi::mouseMoveEvent(QMouseEvent *event) {
    foreach (Widget *w, widgets) {
        if (w->boundingRect.contains(event->pos())) {
            w->mouseMove();

            // 当鼠标在 widget 上移动时，如果 widget 的 hover 为 false，则触发一次 mouseEnter 事件，告知鼠标进入它了
            if (!w->hover) {
                w->mouseEnter();
            }
        } else {
            // 当鼠标不在 widget 上移动时，如果 widget 的 hover 为 true，则触发一次 mouseLeave 事件，告知鼠标已经离开它了
            if (w->hover) {
                w->mouseLeave();
            }
        }
    }

    update();
}

void OSUi::mousePressEvent(QMouseEvent *event) {
    foreach (Widget *w, widgets) {
        if (w->boundingRect.contains(event->pos())) {
            w->mousePressed();
        }
    }

    update();
}

void OSUi::mouseReleaseEvent(QMouseEvent *) {
    foreach (Widget *w, widgets) {
        if (w->pressed) {
            w->mouseReleased();
        }
    }

    update();
}
```

## main

```cpp
#include <QApplication>
#include "AppWidget.h"

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    AppWidget w;
    w.show();
    return a.exec();
}
```

## 思考

1. 上面的程序只绘制了 Button 和 CheckBox，只是最最简单的控件，绘制复杂控件例如 Table 应该要怎么做呢？当需要的控件都一个一个地加入到这个系统里，并能够进行相应的事件响应，我们就创造了一个系统的界面。
2. 更新的时候把所有的控件都绘制了一遍，如果控件很多时效率是不是很低？如果能够只绘制状态改变了的控件是不是就更好了？要实现好这样的算法应该不容易，会涉及到重叠绘制。
3. 鼠标事件发生时选择控件也是遍历了所有的控件，Qt Graphics/View 框架使用 binary space partition 算法来选择控件，大幅的提升了效率，所以它能够高效的处理上百万个图元。
4. 如果控件放在了界面上不可见的地方，是不是就不需要绘制出来了呢？
5. 怎么实现像 QPushButton 的 clicked 信号，响应点击事件？
6. 鼠标事件处理的时候没有把鼠标的坐标信息传给 Widget

还有太多太多的问题，这里就不一一列举了，说了这么多，只是希望大家能够理解系统界面的本质画出来的，不要感觉很神秘，大道至简。

当然道理很简单，但要做好绝不是简单的事，就像原子弹的原理很简单一样：把两块或几块较小的铀的同位素铀-235放进弹头里，周围用黄色炸药包围，只要先引爆炸药，强迫几块较小的铀燃料合并成一块大的，使铀达到可与中子进行链式核反应的程度，就能够引起核爆炸。世界上只有几个有限的国家能够制造原子弹，不是其他国家不知道原理，而是制造工艺跟不上，这就和我们知道了系统界面的原理一个样，但是做不出一个好用的来，原因是各种处理的算法跟不上。