---
title: 右键菜单
date: 2018-03-20 09:51:49
tags: QtBook
---

右键菜单有多种实现方式:

* 设置 contextMenuPolicy 为:
  * Qt::ActionsContextMenu
  * Qt::CustomContextMenu
* 重写 contextMenuEvent 函数

下面就分别介绍这几种右键菜单的实现。

<!--more-->

## ActionsContextMenu

设置 `contextMenuPolicy` 为 `Qt::ActionsContextMenu`，使用 `QWidget::addAction()` 添加 QAction 就实现了右键菜单:

```c++
void MyWidget::createContextMenu() {
    this->setContextMenuPolicy(Qt::ActionsContextMenu);

    this->addAction(new QAction("Java"));
    this->addAction(new QAction("C++"));
    this->addAction(new QAction("PHP"));
    this->addAction(new QAction("JavaScript"));
}
```

代码非常简单，缺点就是菜单项不能根据不同的条件显示和隐藏，优点当然是简单了。

## CustomContextMenu

设置 `contextMenuPolicy` 为 `Qt::CustomContextMenu`，点击右键的时候发射信号 `QWidget::customContextMenuRequested()`，给其绑定一个槽函数，在里面显示菜单即可:

```cpp
void MyWidget::createContextMenu() {
    QAction *javaAction = new QAction("Java", this);
    QAction *cppAction  = new QAction("C++", this);
    QAction *phpAction  = new QAction("PHP", this);
    QAction *jsAction   = new QAction("JavaScript", this);

    this->setContextMenuPolicy(Qt::CustomContextMenu);
    connect(this, &QWidget::customContextMenuRequested, [=](const QPoint &pos) {
        QMenu menu;
        menu.addAction(javaAction);
        menu.addAction(cppAction);
        menu.addAction(phpAction);
        menu.addAction(jsAction);
        menu.exec(mapToGlobal(pos));
    });
}
```

相比较设置 `contextMenuPolicy` 为 `Qt::ActionsContextMenu` 的方式复杂了那么一点点，但好处是可以根据条件显示不同的菜单项。

## contextMenuEvent

QWidget 的 contextMenuPolicy 默认为 `Qt::DefaultContextMenu`，点击右键时调用函数 `contextMenuEvent()`，在里面显示右键菜单:

```c++
void MyWidget::contextMenuEvent(QContextMenuEvent *event) {
    QAction *javaAction = new QAction("Java", this);
    QAction *cppAction  = new QAction("C++", this);
    QAction *phpAction  = new QAction("PHP", this);
    QAction *jsAction   = new QAction("JavaScript", this);

    QMenu menu;
    menu.addAction(javaAction);
    menu.addAction(cppAction);
    menu.addAction(phpAction);
    menu.addAction(jsAction);
    menu.exec(mapToGlobal(event->pos()));
}
```

注意: 实际项目中应该把上面的 action 定义为成员变量，而不是在 `contextMenuEvent()` 函数中创建。

## 响应点击菜单项

上面的右键菜单只是有个样子，点击后没反应，那是因为没有给 action 指定相应的行为。点击菜单项后会发射信号 `triggered()`，给其绑定槽函数就可以做你想做的事了:

```cpp
QAction *javaAction = new QAction("Java", this);

connect(javaAction, &QAction::triggered, [] {
    qDebug() << "点击菜单项";
});
```

## 单选右键菜单

单选右键菜单和 radio button 一样，几个项里只有一个能被选中，把单选的菜单项设置为可选的并加入到一个 QActionGroup 即可:

```cpp
void MyWidget::createContextMenu() {
    // 创建 Action
    QAction *javaAction = new QAction("Java", this);
    QAction *cppAction  = new QAction("C++", this);
    QAction *phpAction  = new QAction("PHP", this);
    QAction *jsAction   = new QAction("JavaScript", this);

    // 设置 Action 为可选
    javaAction->setCheckable(true);
    cppAction->setCheckable(true);
    phpAction->setCheckable(true);
    phpAction->setChecked(true);

    // 需要单选的 Action 添加到 Action Group
    QActionGroup *actionGroup = new QActionGroup(this);
    actionGroup->addAction(javaAction);
    actionGroup->addAction(cppAction);
    actionGroup->addAction(phpAction);

    // 右键时显示菜单
    this->setContextMenuPolicy(Qt::CustomContextMenu);
    connect(this, &QWidget::customContextMenuRequested, [=](const QPoint &pos) {
        QMenu menu;
        menu.addAction(javaAction);
        menu.addAction(cppAction);
        menu.addAction(phpAction);
        menu.addSeparator();
        menu.addAction(jsAction);
        menu.exec(mapToGlobal(pos));
    });
}
```

