---
title: Layout 秘录
date: 2018-02-25 17:44:54
tags: QtBook
---

布局管理器 QHBoxLayout、QVBoxLayout、QGridLayout 相信大家都很熟悉了，对于常用的功能就不一一列举，这里将介绍一下几个不常用，在复杂的自定义界面时又可能会用到的功能:

* QGridLayout 中多个 Widget 放在同一个位置
* 把一个 Widget 替换为另一个 Widget
* QHBoxLayout、QVBoxLayout 中插入 Widget
* 从 Layout 中删除 Widget<!--more-->

## QGridLayout 中多个 Widget 放在同一个位置

什么时候需要把多个 Widget 放在同一个位置呢，例如:

* 给一个 Widget 蒙上一个半透明层，显示正在加载的状态
* 给一个 Widget 右上角加一个关闭按钮，自定义消息提示时可以使用

这种功能需要使用 QGridLayout 把多个 Widget 放在同一个位置，只需要调用 `addWidget()` 时 row 和 column 的值分别相同，并根据不同的需求设置 Widget 的对齐方式:

```cpp
#include <QApplication>
#include <QWidget>
#include <QPushButton>
#include <QGridLayout>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QGridLayout *layout = new QGridLayout();
    QPushButton *button = new QPushButton("X");
    QWidget     *widget = new QWidget();

    layout->setContentsMargins(2, 2, 2, 2);

    // widget 和 button 放在相同的位置，注意它们的添加顺序
    layout->addWidget(widget, 0, 0);
    layout->addWidget(button, 0, 0);

    // 按钮的对齐方式为右上角
    layout->setAlignment(button, Qt::AlignTop | Qt::AlignRight);

    // 设置 QSS 样式
    button->setStyleSheet("border: none; border-radius: 10px;"
                          "background: gray; padding: 0;"
                          "min-width:  20px; max-width:  20px;"
                          "min-height: 20px; max-height: 20px;");
    widget->setStyleSheet("background: #5cadff; margin: 8px;");

    QWidget window;
    window.setLayout(layout);
    window.resize(400, 300);
    window.show();

    return app.exec();
}
```

程序的效果如图:

![](/img/qtbook/custom-widget/layout-top-right-button.png)

## 把一个 Widget 替换为另一个 Widget

在 Ui Designer 里可以使用提升的方式 (Promote to) 把一个 Widget 替换为另一个 Widget (当然实际内部不是替换)，还有一种方式就是使用函数 `QLayout::replace()` 进行替换，常用在把占位用的 Widget 给替换为真正想要显示的 Widget，例如下面的程序点击按钮 `替换` 把占位用的 placeholderLabel 替换为编辑器 editor:

```cpp
#include <QApplication>
#include <QWidget>
#include <QPushButton>
#include <QLabel>
#include <QTextEdit>
#include <QVBoxLayout>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QPushButton *replaceButton = new QPushButton("替换");
    QLabel *placeholderLabel = new QLabel("占位 Label");
    QTextEdit *editor = new QTextEdit();

    QVBoxLayout *layout = new QVBoxLayout();
    layout->addWidget(placeholderLabel);
    layout->addWidget(replaceButton);

    QWidget window;
    window.setLayout(layout);
    window.resize(400, 300);
    window.show();

    // 点击按钮 "替换" 把 placeholderLabel 替换为 editor
    QObject::connect(replaceButton, &QPushButton::clicked, [=] {
        delete layout->replaceWidget(placeholderLabel, editor); // 返回的 QLayoutItem 不需要了
        delete placeholderLabel; // 需要删除或者隐藏
    });

    return app.exec();
}
```

> 注意: 
>
> * `QLayout::replace()` 返回的 QLayoutItem 需要我们自己管理，如果没用了的话直接 delete 掉吧，避免造成内存泄漏，也可以保存起来再次插入其他 layout 中。
> * 被替换下来的 widget 的 parent 还是原来的 parent，也就是说它只不过脱离了 layout 的管理，但还显示在它的 parent 上 (上面的 QTextEdit 换为 QLineEdit 试试就明白了)，如果还有用就保存起来并调用 `QWidget::hide()` 进行隐藏，如果只是用于占位就 delete 掉吧。

## QHBoxLayout、QVBoxLayout 中插入 Widget

例如下面的好友列表使用 QVBoxLayout，点击按钮 `添加` 则会增加一个新的好友到列表里:

![](/img/qtbook/custom-widget/layout-insert.png)

```cpp
#include <QApplication>
#include <QWidget>
#include <QPushButton>
#include <QLabel>
#include <QVBoxLayout>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QPushButton *addButton = new QPushButton("添加");
    QVBoxLayout *layout = new QVBoxLayout();
    layout->addWidget(new QLabel("独孤求败"));
    layout->addWidget(new QLabel("东方不败"));
    layout->addWidget(new QLabel("静寂城主"));
    layout->addStretch();
    layout->addWidget(addButton);

    QWidget window;
    window.setLayout(layout);
    window.resize(130, 300);
    window.show();

    // 点击按钮 "替换" 把 placeHolderLabel 替换为 editor
    QObject::connect(addButton, &QPushButton::clicked, [=] {
        layout->addWidget(new QLabel("新朋友"));
    });

    return app.exec();
}
```

增加新的好友时调用 `layout->addWidget(new QLabel("新朋友"))`，发现被添加到了按钮 `添加` 后面，而我们期望的是添加到最后一个好友后面，这就需要使用 `QLayout::insertWidget()` 把新好友的 QLabel 插入到合适的位置了:

```cpp
layout->addWidget(new QLabel("新朋友"));
替换为
layout->insertWidget(layout->count() - 2, new QLabel("新朋友"));
```

> 提示: 插入的位置为啥是 `layout->count() - 2` 呢？因为 layout 中前面是好友的 QLabel，倒数第二是 stretchable space (`addStretch()`)，最后的位置上是按钮，所以新的好友应该插入到倒数第二的 stretchable space 前面，所以是减去 2。

## 从 Layout 中删除 Widget

从 Layout 中删除 Widget 可以使用下面 2 种方式:

* `widget->deleteLater()`: 会把 widget 从 layout 中删除，也会 delete 和它相关的 QLayoutItem
* `layout->removeWidget(widget)`: 会把 widget 从 layout 中删除，但是 widget 没有被 delete 掉，和它相关的 QLayoutItem 会被 delete 掉

下面的代码作用是清空 Layout (item 有可能没有 widget):

```cpp
void clearLayout(QLayout *layout) {
    QLayoutItem *item;

    while((item = layout->takeAt(0)) != 0) {
        if (item->widget()) {
            delete item->widget();
        } else {
            delete item;
        }
    }
}
```

