---
title: 自定义 Widget 使用 QSS
date: 2017-11-27 22:02:02
tags: QtBook
---

相信很多同学继承如 QWidget，QPushButton 等实现自定义的控件后，发现在此控件上 QSS 不生效了，这不行啊，设置背景、边框、字体等如果没有 QSS 那就太麻烦了。其实在自定义控件上启用 QSS 非常简单，只要调用一下 `setAttribute(Qt::WA_StyledBackground)` 即可，就像下面这样

```cpp
Widget::Widget(QWidget *parent) : QWidget(parent) {
    this->setAttribute(Qt::WA_StyledBackground); // 启用 QSS
    this->setStyleSheet("border: 2px solid red; background: pink; border-radius: 10px;"); // 设置 QSS
}
```

```cpp
#include <QApplication>
#include "Widget.h"

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QWidget window;

    Widget customWidget(&window);
    customWidget.setGeometry(20, 20, 100, 100);

    window.resize(300, 300);
    window.show();

    return app.exec();
}
```

如上在 main() 函数中使用自定义控件 Widget，QSS 生效了，去掉 `this->setAttribute(Qt::WA_StyledBackground)` 后 QSS 就没效果了:

![](/img/qtbook/custom-widget/enable-stylesheet.png)

