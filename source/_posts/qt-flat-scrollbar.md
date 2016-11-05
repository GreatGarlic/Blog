---
title: QSS 实现的扁平滚动条
date: 2016-11-05 12:02:21
tags: Qt
---
使用 QSS 实现扁平滚动条，只有几个简单的颜色，并且去掉了箭头，圆角等，尽量的简约，简约而不简单

* 滚动条的背景色
* handle 的背景色
* 鼠标放到 handle 上的背景色

![](/img/qt/flat-scrollbar.png)

<!--more-->

```css
QScrollBar:vertical {
    width: 8px;
    background: #DDD;
    padding-bottom: 8px;
}

QScrollBar:horizontal {
    height: 8px;
    background: #DDD;
}

QScrollBar::handle:vertical,
QScrollBar::handle:horizontal {
    background: #AAA;
}

QScrollBar::handle:vertical:hover,
QScrollBar::handle:horizontal:hover {
    background: #888;
}

QScrollBar::sub-line:vertical, QScrollBar::add-line:vertical,
QScrollBar::sub-line:horizontal, QScrollBar::add-line:horizontal {
    width: 0;
    height: 0;
}
```

> 实际项目中，根据界面的风格调整颜色，宽高等就可以了
