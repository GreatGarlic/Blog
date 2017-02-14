---
title: 去掉 Windows 中控件的虚线框
date: 2017-02-14 10:39:21
tags: Qt
---
在 Windows 中，控件得到焦点的时候，会显示一个虚线框，很多时候觉得不好看，通过自定义 **QProxyStyle** 就可以把这个虚线框去掉。<!--more-->

```cpp
// 文件名: NoFocusRectStyle.h

#ifndef NOFOCUSRECTSTYLE_H
#define NOFOCUSRECTSTYLE_H

#include <QProxyStyle>

class NoFocusRectStyle : public QProxyStyle {
public:
    NoFocusRectStyle(QStyle *baseStyle) : QProxyStyle(baseStyle) {}

    void drawPrimitive(PrimitiveElement element,
                       const QStyleOption *option,
                       QPainter *painter,
                       const QWidget *widget = 0) const {
        if (element == QStyle::PE_FrameFocusRect) {
            return;
        }

        QProxyStyle::drawPrimitive(element, option, painter, widget);
    }
};

#endif // NOFOCUSRECTSTYLE_H
```
当 element == QStyle::PE_FrameFocusRect 时，直接返回，不绘制虚线框。

```cpp
// 文件名: main.cpp

#include "Widget.h"
#include "NoFocusRectStyle.h"
#include <QApplication>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);
    NoFocusRectStyle *style = new NoFocusRectStyle(app.style());
    app.setStyle(style); // Ownership of the style object is transferred to QApplication

    Widget w;
    w.show();

    return app.exec();
}
```
在 main() 函数中调用 app.setStyle(style) 使用我们上面自定义的 NoFocusRectStyle 就可以把虚线框去掉了。
