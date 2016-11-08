---
title: 鼠标放到 View 的 item 上时显示 tool tip
date: 2016-11-08 09:53:35
tags: Qt
---

```c++
// [1] View 需要启用 mouse tracking
ui->listView->setMouseTracking(true); 

// [2] 处理 mouse enter 事件
connect(ui->listView, &QListView::entered, [] (const QModelIndex &index) {
    if (!index.isValid()) {
        qDebug() << "Invalid index";
        return;
    }

    QToolTip::showText(QCursor::pos(), index.data(Qt::UserRole + 1).toString());
});
```
