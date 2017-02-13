---
title: Qt 显示 Gif
date: 2017-02-13 14:05:39
tags: Qt
---
Qt 中，静态图片 PNG，JPG 等可以用其创建 QPixmap，调用 QLabel::setPixmap() 来显示，但是能够具有动画的 GIF 却不能这么做，要在 QLabel 上显示 GIF，需要借助 QMovie 来实现。<!--more-->

## QLabel 显示 GIF
使用 GIF 图片的路径创建 QMovie 对象，并且调用 `QMovie::start()` 启动 GIF 动画，然后通过 `QLabel::setMovie()` 设置好动画对象后，就能在 QLabel 上看到 GIF 动画了。

```cpp
#include <QApplication>
#include <QMovie>
#include <QLabel>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QLabel *label = new QLabel();
    QMovie *movie = new QMovie("/Users/Biao/Desktop/x.gif");
    label->setMovie(movie); // 1. 设置要显示的 GIF 动画图片
    movie->start();         // 2. 启动动画
    label->show();
    
    return app.exec();
}
```

## 控制 GIF 动画循环次数
```cpp
#include <QApplication>
#include <QMovie>
#include <QLabel>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QLabel *label = new QLabel();
    QMovie *movie = new QMovie("/Users/Biao/Desktop/x.gif");
    label->setMovie(movie); // 1. 设置要显示的 GIF 动画图片
    movie->start();         // 2. 启动动画
    label->show();

    QObject::connect(movie, &QMovie::frameChanged, [=](int frameNumber) {
        // GIF 动画执行一次就结束
        if (frameNumber == movie->frameCount() - 1) {
            movie->stop();
        }
    });

    return app.exec();
}
```
如果查看 QMovie API，会发现 `QMovie::loopCount()` 能够返回 GIF 动画的循环次数，但是这个次数是创建 GIF 时设置的次数，QMovie 没有提供 API 来设置动画循环的次数，不过我们可以监听动画执行时的 frameChanged 信号，如果当前帧是 GIF 的最后一帧时则说明一次动画播放完成，需要停止动画播放时调用 `QMovie::stop()` 即可。上面的例子中 GIF 动画执行完一次的时候就停止，如果需要执行完 3 次时才停止，应该怎么修改呢？

QMovie 中 GIF 动画帧的序号从 0 开始计数，例如共有 200 帧的 GIF 动画的最后一帧的下标是 199，`QMovie::frameCount()` 返回 GIF 的帧数。
