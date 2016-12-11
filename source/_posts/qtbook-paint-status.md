---
title: QPainter 的状态保存与恢复
date: 2016-12-08 23:21:50
tags: QtBook
---
实现这样的一个程序，把 QPainter 的坐标原点从左上角移动到 (100, 100)，然后画出坐标轴，接下来顺时针旋转坐标轴 45 度，设置画笔，画刷，字体，画一个矩形和字符串，最后恢复 QPainter 到最开始的状态，即还原画笔，画刷，字体，逆时针旋转坐标轴 45 度，移动 QPainter 的坐标原点到左上角，再画一个矩形和字符串，就像下图这样：

![](/img/qtbook/paint/Paint-Base-SaveRestore.png)

<!--more-->

再不了解 QPainter 的 save() 和 restore() 之前，我们可能会下面这样做：

```cpp
void CodeStatusWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);

    // [*] 保存 QPainter 的状态
    QPen pen = painter.pen();
    QBrush brush = painter.brush();
    QFont font = painter.font();

    // 移动坐标轴
    painter.translate(100, 100);

    // 绘制坐标轴所在的线
    painter.drawLine(-100, 0, 100, 0);
    painter.drawLine(0, -100, 0, 100);

    painter.rotate(45);
    painter.setPen(Qt::red);
    painter.setBrush(Qt::blue);
    painter.setFont(QFont("Monaco", 30));

    painter.drawRect(-50, -50, 100, 100);
    painter.drawText(0, 0, "Hello");

    // [*] 恢复 QPainter 的状态
    painter.rotate(-45);
    painter.translate(-100, -100);
    painter.setFont(font);
    painter.setPen(pen);
    painter.setBrush(brush);

    painter.drawRect(250, 50, 100, 100);
    painter.drawText(250, 50, "Hello");
}
```

为了能够恢复 QPainter 的状态，首先要保存 QPainter 的状态

```cpp
    QPen pen = painter.pen();
    QBrush brush = painter.brush();
    QFont font = painter.font();
```

最后使用下面的代码恢复 QPainter 的状态

```cpp
    painter.rotate(-45);
    painter.translate(-100, -100);
    painter.setFont(font);
    painter.setPen(pen);
    painter.setBrush(brush);
```

看上去不是很难，但是在绘制复杂图形的时候，恢复 QPainter 状态的操作就会很频繁，很有可能导致代码难以管理。如果仔细阅读 QPainter 的帮助文档，就会发现其实 QPainter 已经提供了接口用于保存和恢复它的状态，就是 save() 和 restore()。

> `void QPainter::save()` - Saves the current painter state (pushes the state onto a stack). A save() must be followed by a corresponding restore().
>
> `void QPainter::restore()` - Restores the current painter state (pops a saved state off the stack).

save() 用于保存 QPainter 的状态，restore() 用于恢复 QPainter 的状态，save() 和 restore() 一般都是成对使用的，如果只调用了 save() 而不调用 restore()，那么保存就没有意义了，保存是为了能恢复被保存的状态而使用的。QPainter 的状态有画笔，画刷，字体，变换（旋转，移动，切变，缩放）等。

下面就使用 QPainter 提供的功能保存和恢复它的状态，看看代码会是怎么样的

```cpp
void ApiStatusWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);

    // [*] 保存 QPainter 的状态
    painter.save();

    // 移动坐标轴
    painter.translate(100, 100);

    // 绘制坐标轴所在的线
    painter.drawLine(-100, 0, 100, 0);
    painter.drawLine(0, -100, 0, 100);

    painter.rotate(45);
    painter.setPen(Qt::red);
    painter.setBrush(Qt::blue);
    painter.setFont(QFont("Monaco", 30));

    painter.drawRect(-50, -50, 100, 100);
    painter.drawText(0, 0, "Hello");

    // [*] 恢复 QPainter 的状态
    painter.restore();

    painter.drawRect(250, 50, 100, 100);
    painter.drawText(250, 50, "Hello");
}
```

利用 save() 和 restore() 后，就不需要开始的时候一个一个的保存和恢复 QPainter 的状态了，调用一下 save()，QPainter 的所有状态就保存好了，需要恢复的时候只要调用 restore() 所有保存好的状态就恢复回来了，代码简洁了很多，也不会不小心漏掉某些状态而出错。

此外，save() 和 restore() 可以以堆栈的形式嵌套式地保存和恢复，最后保存的先恢复：

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);

    painter.save(); // 保存状态 1
    ...
    painter.save(); // 保存状态 2
    ...
    painter.save();    // 保存状态 3
    ...
    painter.restore(); // 恢复状态 3
    ...
    painter.save();    // 保存状态 4
    ...
    painter.restore(); // 恢复状态 4
    painter.restore(); // 恢复状态 2
    ...
    painter.restore(); // 恢复状态 1
}
```

像上面这样复杂状态的恢复，如果像开始那样我们自己写代码实现，想想就很可怕。
