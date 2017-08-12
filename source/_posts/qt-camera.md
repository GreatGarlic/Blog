---
title: Qt 调用摄像头
date: 2017-08-08 10:33:40
tags: Qt
---

可以使用 OpenCV 来操作摄像头，不过 Qt5 已经自带了调用系统摄像头的功能，使用起来很方便，主要是使用下面 3 个类:

* QCamera
* QCameraViewfinder
* QCameraImageCapture

下面代码的效果为

![](/img/qt/camera.png)

<!--more-->

```cpp
#include <QApplication>
#include <QDebug>
#include <QCamera>
#include <QCameraViewfinder>
#include <QCameraImageCapture>
#include <QImage>
#include <QPixmap>
#include <QVBoxLayout>
#include <QPushButton>
#include <QLabel>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    a.setStyleSheet("QCameraViewfinder, QLabel { border: 2px dashed grey;}");

    QCamera *camera = new QCamera(); // 摄像头对象
    QCameraViewfinder *viewfinder = new QCameraViewfinder(); // 用于实时显示摄像头图像
    QCameraImageCapture *imageCapture = new QCameraImageCapture(camera); // 用于截取摄像头图像
    camera->setViewfinder(viewfinder); // camera 使用 viewfinder 实时显示图像
    viewfinder->setAttribute(Qt::WA_StyledBackground, true); // 使 viewfinder 能够使用 QSS

    QLabel *previewLabel = new QLabel(""); // 拍照预览的 label
    QPushButton *captureButton = new QPushButton("拍照"); // 点击拍照的按钮

    viewfinder->setFixedHeight(150);
    previewLabel->setFixedHeight(150);
    previewLabel->setMinimumWidth(150);
    previewLabel->setAlignment(Qt::AlignCenter);

    // 布局 widgets
    QVBoxLayout *layout = new QVBoxLayout();
    layout->addWidget(viewfinder);
    layout->addWidget(previewLabel);
    layout->addWidget(captureButton);
    layout->addStretch();

    QWidget *window = new QWidget();
    window->setLayout(layout);
    window->show();
    camera->start();

    // 点击拍照
    QObject::connect(captureButton, &QPushButton::clicked, [=] {
        imageCapture->capture("capture.jpg"); // 如果不传入截图文件的路径，则会使用默认的路径，每次截图生成一个图片
    });

    // 拍照完成后的槽函数，可以把 image 保存到自己想要的位置
    QObject::connect(imageCapture, &QCameraImageCapture::imageCaptured, [=](int id, const QImage &image) {
        Q_UNUSED(id)
        QImage scaledImage = image.scaled(previewLabel->size(), Qt::KeepAspectRatio, Qt::SmoothTransformation);
        previewLabel->setPixmap(QPixmap::fromImage(scaledImage));
    });

    return a.exec();
}
```

**QCamera *camera = new QCamera()** 初始化摄像头对象，调用 **camera->start()** 打开摄像头，成功打开后可以在 **QCameraViewfinder** 上实时的看到摄像头的图像，如果打开失败则什么都看不到。

**QCameraImageCapture::capture()** 用于捕捉摄像头的图像，完成后会发射信号 **imageCaptured()**，可以在这个信号的槽函数中把捕捉到的图像显示给用户，如果满意的话就可以保存到文件，或者上传到服务器等。**capture()** 接受一个参数，为截图后图片保存的路径，如果使用默认的路径，每次截图都会生成一个图片，如果想截图产生的图片都保存为一个固定的图片文件，则传入一个路径即可。

仔细的看一下可以发现 **QImage::scaled()** 缩放的图像和 **QCameraViewfinder** 上缩放的图像的效果不一样，这是因为缩放的算法不同导致的，如果不满足需求，可以使用更专业的图像缩放算法进行处理。

需要注意的是 **QCameraViewfinder** 的 size policy 很奇怪，vertical 和 horizontal 都是 Preferred，和 QLabel 默认的 size policy 一样，但是表现出来的却是 expanding 的样子，甚至有 spacer 时，spacer 的 expanding 都没有效果，空间全被 **QCameraViewfinder** 占据了。

