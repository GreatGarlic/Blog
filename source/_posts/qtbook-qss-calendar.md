---
title: QSS QCalendarWidget
date: 2017-02-03 17:40:31
tags: QtBook
---
QCalendarWidget 是一个比较复杂的 widget，由几个 QToolButton, QSpinBox, QMenu, QTableView 等组成，Qt 的帮助文档里没有其 QSS 的相关文档，当要修改其样式的时候应该怎么办呢？

我们这里采用的方法是分析组成 QCalendarWidget 的 widget 的 className 和 objectName，然后 QSS 每个 widget，最终达到修改 QCalendarWidget 样式的目的。<!--more-->

```cpp
MainWidget::MainWidget(QWidget *parent) : QWidget(parent), ui(new Ui::MainWidget) {
    ui->setupUi(this);
    dumpStructure(ui->calendarWidget, 0);
}

MainWidget::~MainWidget() {
}

void MainWidget::dumpStructure(const QObject *obj, int spaceCount) {
    qDebug() << QString("%1%2 : %3")
                .arg("", spaceCount)
                .arg(obj->metaObject()->className())
                .arg(obj->objectName());

    QObjectList list = obj->children();

    foreach (QObject * child, list) {
        dumpStructure(child, spaceCount + 4);
    }
}
```

使用函数 `dumpStructure()` 打印出 QCalendarWidget 的树形组成结构，输出如下：

```
"QCalendarWidget : calendarWidget"
"    QVBoxLayout : "
"    QCalendarModel : "
"    QCalendarView : qt_calendar_calendarview"
"        QWidget : qt_scrollarea_viewport"
"        QWidget : qt_scrollarea_hcontainer"
"            QScrollBar : "
"            QBoxLayout : "
"        QWidget : qt_scrollarea_vcontainer"
"            QScrollBar : "
"            QBoxLayout : "
"        QStyledItemDelegate : "
"        QHeaderView : "
"            QWidget : qt_scrollarea_viewport"
"            QWidget : qt_scrollarea_hcontainer"
"                QScrollBar : "
"                QBoxLayout : "
"            QWidget : qt_scrollarea_vcontainer"
"                QScrollBar : "
"                QBoxLayout : "
"            QItemSelectionModel : "
"        QHeaderView : "
"            QWidget : qt_scrollarea_viewport"
"            QWidget : qt_scrollarea_hcontainer"
"                QScrollBar : "
"                QBoxLayout : "
"            QWidget : qt_scrollarea_vcontainer"
"                QScrollBar : "
"                QBoxLayout : "
"            QItemSelectionModel : "
"        QTableCornerButton : "
"        QItemSelectionModel : "
"    QWidget : qt_calendar_navigationbar"
"        QPrevNextCalButton : qt_calendar_prevmonth"
"        QPrevNextCalButton : qt_calendar_nextmonth"
"        QToolButton : qt_calendar_monthbutton"
"            QMenu : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"                QAction : "
"        QToolButton : qt_calendar_yearbutton"
"        QSpinBox : qt_calendar_yearedit"
"            QLineEdit : qt_spinbox_lineedit"
"                QWidgetLineControl : "
"            QValidator : qt_spinboxvalidator"
"        QHBoxLayout : "
"    QCalendarDelegate : "
"    QCalendarTextNavigator : "
```

分析上面输出的 objectName，不难得出它们对应的 widget 如下图所示：  
![](/img/qtbook/qss/QSS-Subcontrol-CalendarWidget.png)

知道了每个 widget 后，就可以像下面这样用 QSS 修改 QCalendarWidget 的样式了。

```css
#qt_calendar_calendarview {
    background: white;
}

#qt_calendar_navigationbar {
    background: rgba(215, 215, 215, 255);
}

QToolButton {
    icon-size: 30px, 30px;
    width: 80px;
    height: 30px;
}

QToolButton#qt_calendar_prevmonth {
    background: green;
    width: 40px;
    qproperty-icon: url(:/resources/tabset-left.png);
}

QToolButton#qt_calendar_nextmonth {
    background: blue;
    width: 40px;
    qproperty-icon: url(:/resources/tabset-right.png);
}

QToolButton#qt_calendar_monthbutton {
    background: yellow;
    padding-right: 20px;
}

QToolButton#qt_calendar_yearbutton {
    background: magenta;
}

QToolButton#qt_calendar_monthbutton::menu-indicator{
    subcontrol-origin: padding;
    subcontrol-position: center right;
    right: 3px;
    width: 10px;
}

QAbstractItemView {
    color: black;
    selection-color: white;
    selection-background-color: rgb(255, 174, 0);
    font: 15px;
}
```

得到效果：  
![](/img/qtbook/qss/QSS-Subcontrol-CalendarWidget-1.png)

但是还有一些问题，QSS 对 QCalendar 里的 QHeaderView::section 和 QTableView::item 没有效果，看到有人说可以用 QPalette 修改其颜色和背景，如有兴趣的话可以自己试试。

虽然 QCalendarWidget 的 QSS 还有很多小问题，就不在继续往下说了，这里主要是抛砖引玉，提出一种方法解决复杂甚至未知 widget 的 QSS，根据这个思路，举一反三，在遇到相似问题的时候，大家应该都能有个方向了。

