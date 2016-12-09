---
title: 画刷
date: 2016-12-08 23:12:40
tags: QtBook
---
画刷 QBrush 是用来填充图形用的

> The QBrush class defines the fill pattern of shapes drawn by QPainter.  
> A brush has a style, a color, a gradient and a texture.

下图在来自 QBrush 的帮助文档内容，列出了 Qt 自带的 brush：

![](/img/qtbook/paint/Paint-Base-Brush.png)

画刷还可以使用 QPixmap，渐变等来创建。

平铺绘制 QPixmap 可以使用 QPainter::drawTiledPixmap()，也可以像下面这样使用 texture pattern 实现平铺绘制:

```cpp
void MainWidget::paintEvent(QPaintEvent *) {
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);

    QPixmap pixmap(":/resources/Paint-Base-Bufferfly.png");
    QBrush brush(pixmap);
    painter.setBrush(brush);
    painter.drawRect(0, 0, width(), height());
}
```
