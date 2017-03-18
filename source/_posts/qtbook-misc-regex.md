---
title: 实用正则表达式
date: 2017-02-24 15:28:14
tags: Qt
---
Qt 里正则表达式使用 **QRegularExpression**，可以使用正则表达式查找字符串，QString 中可以使用正则表达式 **QRegularExpression** 进行字符串替换，拆分等。<!--more-->

## 查找字符串中的 URL:

```cpp
#include <QDebug>
#include <QRegularExpression>

int main(int argc, char *argv[]) {
    // [1] 简单的 URL 正则表达式
    QRegularExpression regExp("http://.+?\\.(com\\.cn|com)");

    // [2.1] 查找第一个 URL
    QRegularExpressionMatch match1 = regExp.match("请把http://www.baidu.com和http://www.sina.com.cn打开看看");
    qDebug() << match1.captured(0);

    // [2.2] 查找所有的 URL
    QRegularExpressionMatchIterator iter = regExp.globalMatch("请把http://www.baidu.com和http://www.sina.com.cn打开看看");

    while (iter.hasNext()) {
        QRegularExpressionMatch match2 = iter.next();
        qDebug() << match2.captured(0);
    }

    return 0;
}
```
输出

```
"http://www.baidu.com"
"http://www.baidu.com"
"http://www.sina.com.cn"
```

## 字符串替换
```cpp
#include <QDebug>
#include <QRegularExpression>

int main(int argc, char *argv[]) {
    // [1] 普通替换
    QString s = "Banana";
    s.replace(QRegularExpression("a[mn]"), "ox");
    qDebug() << s; // s == "Boxoxa"

    // [2] 替换时使用正则捕捉到的第一组的内容, \\1 表示第一组
    QString t = "A <i>bon mot</i>.";
    t.replace(QRegularExpression("<i>([^<]*)</i>"), "\\emph{\\1}");
    qDebug() << t; // t == "A \\emph{bon mot}."

    return 0;
}
```

## 字符串拆分
```cpp
#include <QDebug>
#include <QRegularExpression>

int main(int argc, char *argv[]) {
    QString str = "Some  text\n\twith  strange whitespace.";;
    QStringList list = str.split(QRegularExpression("\\s+")); // 用连续的多个空白字符进行字符串拆分，空格，回车等都是空白字符
    qDebug() << list; // list: [ "Some", "text", "with", "strange", "whitespace." ]

    return 0;
}
```
