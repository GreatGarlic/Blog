---
title: QLineEdit 中增加按钮
date: 2018-07-15 14:16:02
tags: QtBook
---

下图为 Safari 的地址栏，在输入框右边有一个刷新按钮:

![](/img/qtbook/custom-widget/lineedit-with-button-1.png)

在输入框中增加按钮的设计是比较常见的，例如 Chrome 的地址栏、Firefox 的搜索框、Tim 的搜索框等，这种控件 Qt 没有提供，那应该怎么实现呢？下面提供 2 种思路:

* QLineEdit + QPushButton 使用 QHBoxLayout 布局到一个 QWidget 中，去掉 QLineEdit 得到焦点时的高亮效果，此时应该高亮它的父控件，失去焦点时取消它的父控件的高亮效果
* QLineEdit 作为一个普通的 QWidget，也就是它能够使用 QLayout 把 QPushButton 作为子控件布局到它里面

思路有了，第一种方式需要写很多代码进行控制，实现比较麻烦，下面只介绍第二种思路的实现。<!--more-->

QHBoxLayout 布局这种简单的活我们就不进行介绍了，使用下面的代码就能给 QLineEdit 右边增加一个按钮:

```cpp
#include <QApplication>
#include <QLineEdit>
#include <QHBoxLayout>
#include <QPushButton>
#include <QDebug>

/**
 * 在 QLineEdit 的右边增加一个按钮
 */
QPushButton* createLineEditRightButton(QLineEdit *edit) {
    QPushButton *button = new QPushButton();
    QHBoxLayout *layout = new QHBoxLayout();
    button->setCursor(Qt::ArrowCursor);
    layout->addStretch();
    layout->addWidget(button);
    layout->setContentsMargins(0, 0, 0, 0);
    edit->setLayout(layout);

    return button;
}

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    QWidget widget;
    QLineEdit *edit = new QLineEdit(&widget);
    QPushButton *button = createLineEditRightButton(edit);
    edit->setGeometry(20, 20, 200, 26); // 为了简单起见，没有用布局，直接设置 geometry
    widget.resize(400, 100);
    widget.show();

    QObject::connect(button, &QPushButton::clicked, [] {
        qDebug() << "点击按钮";
    });

    return a.exec();
}
```

运行程序，效果如图:

![](/img/qtbook/custom-widget/lineedit-with-button-2.png)

功能实现了，能够进行输入，也能够点解按钮执行对应的槽函数，但是这效果和文章开头处 Safari 的地址栏比较，很不专业，如果你要是把这效果交给客户，估计我就有赚外快的机会了。

不就是丑吗，有什么大不了的，有了 QSS，美化还不就是甩甩手的事么:

```css
QLineEdit {
    border: 1px solid #EEE;
    border-radius: 4px;
    padding-right: 14px;
}

QLineEdit:focus {
    border-color: #bbbec4;
}

QLineEdit QPushButton {
    width:  16px;
    height: 16px;
    qproperty-flat: true;
    margin-right: 4px;
    border: none;
    border-width: 0;
    border-image: url(/Users/Biao/Desktop/refresh.png) 0 0 0 0 stretch stretch;
}
```

得到的效果如图，是不是好了很多:

![](/img/qtbook/custom-widget/lineedit-with-button-3.png)

怎么使用 QSS 就不用再多说了吧，还有就是自己想一下，怎么给 QLineEdit 的左边增加一个图标呢？