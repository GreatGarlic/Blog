---
title: 拖拽鼠标画矩形
date: 2017-11-19 17:14:51
tags: QtBook
---

经常看到有同学问：如何实现用鼠标拖拽出一个矩形区域的效果，类似 QQ 截图的那个矩形区域？

很简单的一个问题，主要是处理鼠标的按下、移动、松开三个事件:

* 按下鼠标：用一个变量标记要开始拖拽出矩形了，并记录按下的位置作为矩形的左上角顶点
* 移动鼠标：按下时移动鼠标，鼠标的当前位置作为矩形新的右下角的顶点，每次移动事件发生时都要重新画一次矩形，因为矩形变了
* 松开鼠标：清除拖拽矩形的标志，松开鼠标后，移动鼠标时就不要再改变矩形了<!--more-->

```cpp
#ifndef MOUSESELECTIONWIDGET_H
#define MOUSESELECTIONWIDGET_H

#include <QWidget>

class MouseSelectionWidget : public QWidget {
    Q_OBJECT

public:
    explicit MouseSelectionWidget(QWidget *parent = 0);

protected:
    void paintEvent(QPaintEvent *event) Q_DECL_OVERRIDE;
    void mousePressEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mouseReleaseEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mouseMoveEvent(QMouseEvent *event) Q_DECL_OVERRIDE;

private:
    QRect rect;       // 鼠标拖拽出的矩形
    bool  modifyMode; // 为 true 表示编辑矩形模式，为 false 表示不再修改矩形模式
};

#endif // MOUSESELECTIONWIDGET_H
```

```cpp
#include "MouseSelectionWidget.h"
#include <QMouseEvent>
#include <QPainter>
#include <QDebug>

MouseSelectionWidget::MouseSelectionWidget(QWidget *parent) : QWidget(parent), modifyMode(false) {
}

void MouseSelectionWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.drawRect(rect);
}

void MouseSelectionWidget::mousePressEvent(QMouseEvent *event) {
    modifyMode = true;
    rect.setTopLeft(event->pos());
}

void MouseSelectionWidget::mouseReleaseEvent(QMouseEvent *) {
    modifyMode = false;
}

void MouseSelectionWidget::mouseMoveEvent(QMouseEvent *event) {
    if (modifyMode) {
        rect.setBottomRight(event->pos());
        update();
    }
}
```

鼠标按下时，第 15 行设置矩形的左上角坐标，鼠标移动时第 25 行设置鼠标的右下角坐标，并且调用 `update()` 请求刷新界面，最后 Qt 会调用 `paintEvent()` 重新画矩形，看到拖拽时变化的矩形。

---

**update() 和 repaint() 的区别:**

update() 和 repaint() 都会调用 paintEvent() 刷新界面，不过它们有些不同。repaint() 会立即调用 paintEvent()，但是 update() 不会立即调用 paintEvent()，而是在下一个事件循环时调用 paintEvent()，此外 Qt 会自动合并多个 update() 事件减少不必要的 paintEvent() 调用降低闪烁的可能性和提高效率。

粗暴的理解就是 1000 次 update() 有可能只会调用 99 次 paintEvent()，而 1000 次 repaint() 会调用 1000 次 paintEvent()。

绝大多数时候调用 update() 更新界面就够了，什么时候使用 repaint() 呢？Qt 的帮助文档如下说:

> We suggest only using repaint() if you need an immediate repaint, for example during animation. In almost all circumstances update() is better, as it permits Qt to optimize for speed and minimize flicker.
>
> **Warning:** If you call repaint() in a function which may itself be called from paintEvent(), you may get infinite recursion. The update() function never causes recursion.

## 思考

知道了怎么实现拖拽鼠标画矩形，那么想一下，怎么实现拖拽鼠标画出鼠标移动的轨迹呢，甚至多次点击鼠标怎么连成折线呢？