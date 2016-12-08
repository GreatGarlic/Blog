---
title: QPainter 的状态保存与恢复
date: 2016-12-08 23:21:50
tags: Qt-Book
---
save() 用于保存 QPainter 的状态，restore() 用于恢复 QPainter 的状态，save() 和 restore() 一般都是成对使用的，如果只调用了 save() 而不调用 restore()，那么保存就没有意义了，保存是为了能恢复被保存的状态而使用的。QPainter 的状态有画笔，画刷，字体，变换（旋转，移动，切变，缩放）等。

> `void QPainter::save()` - Saves the current painter state (pushes the state onto a stack). A save() must be followed by a corresponding restore().
>
> `void QPainter::restore()` - Restores the current painter state (pops a saved state off the stack). <!--more-->

使用 save() 和 restore() 有什么好处呢？例如我们要写这么一个程序，把 QPainter 的坐标中心从左上角移动到 (100, 100)，然后画出坐标轴，接下来顺时针选择坐标轴 45 度，设置画笔，画刷，字体，画一个矩形和字符串，最后恢复 QPainter 到最开始的状态，再画一个矩形和字符串，就像下图这样：

![](/img/qtbook/paint/Paint-Base-SaveRestore.png)

如果我们不知道 QPainter 提供了 save() 和 restore() 来保存和恢复它的状态，那么就只好定义变量一个一个的保存好所有的状态，需要的时候再逐一恢复：

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);

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

    // 恢复 QPainter 的状态
    painter.rotate(-45);
    painter.translate(-100, -100);
    painter.setFont(font);
    painter.setPen(pen);
    painter.setBrush(brush);

    painter.drawRect(250, 50, 100, 100);
    painter.drawText(250, 50, "Hello");
}
```

不过，现在我们知道了 save() 和 restore()，就不需要像上面这样一个一个的保存和恢复 QPainter 的状态了，调用一下 save() 所有的状态就保存好了，是不是比使用那么多变量犀利了很多？需要恢复的时候只要调用 restore() 所有保存好的状态都恢复回来了，实现同样的功能，代码简洁了很多，也不会不小心漏掉某些状态而出错：

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);

    // 保存 QPainter 的状态
    painter.save();
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

    // 恢复 QPainter 的状态
    painter.restore();

    painter.drawRect(250, 50, 100, 100);
    painter.drawText(250, 50, "Hello");
}
```

此外，save() 和 restore() 可以调用多个的，是以堆栈的形式保存和恢复的，最后保存的先恢复：

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);

    painter.save(); // 保存状态 1
    ...
    painter.save(); // 保存状态 2
    ...
    painter.save(); // 保存状态 3

    painter.restore(); // 恢复状态 3
    ...
    painter.restore(); // 恢复状态 2
    ...
    painter.restore(); // 恢复状态 1
}
```

利用 QPainter::save() 和 QPainter::restore() 可以使得我们的代码更简洁，可读性更高。
