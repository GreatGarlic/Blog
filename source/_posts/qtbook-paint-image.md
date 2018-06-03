---
title: 操作图像像素，实现各种效果
date: 2018-06-03 22:03:39
tags: QtBook
---

Qt 中图像相关的类主要是 QPixmap 和 QImage，QPixmap 没有提供访问图像像素数据的接口，访问图像的像素数据需要使用 QImage，主要的函数有 (相关重载函数没有列出来):

```cpp
// 获取图像的像素数据
QRgb   pixel(int x, int y) const
QColor pixelColor(int x, int y) const
uchar* scanLine(int i)

// 设置图像的像素数据
void setPixel(int x, int y, uint index_or_rgb)
void setPixelColor(int x, int y, const QColor &color)
```

下面把一个图像转为灰度图为例介绍怎么操作图像的像素:

1. 取得图像的宽、高
2. 根据宽、高遍历每一个像素
3. 得到每一个像素的 RGBA 颜色分量
4. 对得到的颜色分量 RGBA 进行灰度计算得到新的颜色
5. 使用计算得到的颜色设置对应像素<!--more-->

```cpp
QImage gray(QImage image) {
    // [1] 获取图像的宽和高
    int w = image.width();
    int h = image.height();

    // [2] 遍历图像的每一个像素
    for (int y = 0; y < h; ++y) {
        for (int x = 0; x < w; ++x) {
            // [3] 获取像素的 RGBA 颜色分量
            QRgb color = image.pixel(x, y);
            int r = qRed(color);
            int g = qGreen(color);
            int b = qBlue(color);
            int a = qAlpha(color);

            // [4] 计算灰色的颜色: 3 个颜色分量的平均值
            int rr = (r + g + b) / 3;
            int rg = rr;
            int rb = rr;
            color = qRgba(rr, rg, rb, a);
            
            // [5] 设置图像的像素
            image.setPixel(x, y, color);
        }
    }

    return image;
}
```

> 参数为 QImage 而不是 const QImage &，使用了 Qt 的隐式共享特性，当 QImage 的内容变化后会复制创建一个新的 QImage，不会影响原来的图像。

操作图像的像素数据是不是一点也不难，接下来实现红色蒙版效果，让图像呈现一种偏红的效果，算法是将红色通道设为红、绿、蓝三个值的平均值，而将绿色通道和蓝色通道都设为 0，参考上面的代码，很快就完成了:

```cpp
QImage red(QImage image) {
    // [1] 获取图像的宽和高
    int w = image.width();
    int h = image.height();

    // [2] 遍历图像的每一个像素
    for (int y = 0; y < h; ++y) {
        for (int x = 0; x < w; ++x) {
            // [3] 获取像素的 RGBA 颜色分量
            QRgb color = image.pixel(x, y);
            int r = qRed(color);
            int g = qGreen(color);
            int b = qBlue(color);
            int a = qAlpha(color);

            // [4] 计算红色蒙版效果
            int rr = (r + g + b) / 3;
            int rg = 0;
            int rb = 0;
            color = qRgba(rr, rg, rb, a);

            // [5] 设置图像的像素
            image.setPixel(x, y, color);
        }
    }

    return image;
}
```

接下来大家自己实现一下亮度效果，让图像变得更亮或更暗，算法是将红色通道、绿色通道、蓝色通道，同时加上一个正值或负值。有了上面的经验，相信我们很快就能写出来。

到此我们虽然学会了操作图像的像素实现各种效果，但是仔细观察它们的代码，会发现 [1], [2], [3], [5] 部分的代码都是重复的，只有实现不同效果的部分 [4] 不同。

代码重用一直是不变的主题，对于上面的实现，可以把不变的部分封装到一个函数中，变化的部分使用函数指针传进来，实现代码重用，其实这就是使用了设计模式中的策略模式，优化后的代码如下:

* 函数 map 封装了不变的部分

  map 不是地图，而是映射的意思，把一个像素的颜色映射成另一个颜色

* 实现图像特效的部分使用 lambda 函数作为参数传给函数 map

```cpp
void map(QImage *image, std::function<QRgb (int r, int g, int b, int a)> process) {
    // [1] 获取图像的宽和高
    int w = image->width();
    int h = image->height();

    // [2] 遍历图像的每一个像素
    for (int y = 0; y < h; ++y) {
        for (int x = 0; x < w; ++x) {
            // [3] 获取像素的 RGBA 颜色分量
            QRgb color = image->pixel(x, y);
            int r = qRed(color);
            int g = qGreen(color);
            int b = qBlue(color);
            int a = qAlpha(color);

            // [4] 计算效果的颜色，这里是变化的部分
            color = process(r, g, b, a);
            
            // [5] 设置图像的像素
            image->setPixel(x, y, color);
        }
    }
}
```

使用函数 map，灰度效果和红色蒙版效果的代码精简如下:

```cpp
QImage gray(QImage image) {
    map(&image, [](int r, int g, int b, int a) -> QRgb {
        int rr = (r + g + b) / 3;
        int rg = rr;
        int rb = rr;

        return qRgba(rr, rg, rb, a);
    });

    return image;
}

QImage red(QImage image) {
    map(&image, [](int r, int g, int b, int a) -> QRgb {
        int rr = (r + g + b) / 3;
        int rg = 0;
        int rb = 0;

        return qRgba(rr, rg, rb, a);
    });

    return image;
}
```

实现高亮效果的代码则如下:

```cpp
QImage brightness(QImage image, int delta) {
    map(&image, [=](int r, int g, int b, int a) -> QRgb {
        int rr = qMax(0, qMin(255, r + delta));
        int rg = qMax(0, qMin(255, g + delta));
        int rb = qMax(0, qMin(255, b + delta));

        return qRgba(rr, rg, rb, a);
    });

    return image;
}
```

使用策略模式后，实现各种效果的代码变的非常简短、漂亮了很多，也减少了出错的几率，非常有价值。

最后附上图像效果的实现类 `ImageEffects`，希望大家可以把它完善的更好，增加更多的效果:

```cpp
// 文件名: ImageEffects.h

#ifndef IMAGEEFFECTS_H
#define IMAGEEFFECTS_H
#include <QImage>
#include <functional>

class ImageEffects {
public:
    /**
     * 灰度效果
     * 灰度效果(grayscale)就是取红、绿、蓝三个像素值的算术平均值，这实际上将图像转成了黑白形式。
     *
     * @param image 原始图片
     * @return 返回新的效果图
     */
    static QImage gray(QImage image);

    /**
     * 复古效果
     * 复古效果(sepia)则是将红、绿、蓝三个像素，分别取这三个值的某种加权平均值，使得图像有一种古旧的效果。
     *
     * @param image 原始图片
     * @return 返回新的效果图
     */
    static QImage siepa(QImage image);

    /**
     * 红色蒙版
     * 红色蒙版指的是，让图像呈现一种偏红的效果。算法是将红色通道设为红、绿、蓝三个值的平均值，而将绿色通道和蓝色通道都设为 0。
     *
     * @param image 原始图片
     * @return 返回新的效果图
     */
    static QImage red(QImage image);

    /**
     * 反转效果
     * 反转效果(invert)是指图片呈现一种色彩颠倒的效果。算法为红、绿、蓝通道都取各自的相反值(255-原值)。
     *
     * @param image 原始图片
     * @return 返回新的效果图
     */
    static QImage invert(QImage image);

    /**
     * 亮度效果
     * 亮度效果(brightness)是指让图像变得更亮或更暗。算法将红色通道、绿色通道、蓝色通道，同时加上一个正值或负值。
     *
     * @param image 原始图片
     * @return 返回新的效果图
     */
    static QImage brightness(QImage image, int delta);

    /**
     * 把图片上每一个像素的颜色映射为函数 process() 的计算结果。
     *
     * @param image   要进行变换的图片指针
     * @param process 对每一个像素的颜色进行计算的函数
     */
    static void map(QImage *image, std::function<QRgb (int r, int g, int b, int a)> process);
};

#endif // IMAGEEFFECTS_H
```

```cpp
// 文件名: ImageEffects.cpp

#include "ImageEffects.h"

// 灰度效果(grayscale)就是取红、绿、蓝三个像素值的算术平均值，这实际上将图像转成了黑白形式。
QImage ImageEffects::gray(QImage image) {
    map(&image, [](int r, int g, int b, int a) -> QRgb {
        int rr = (r + g + b) / 3; // qGray(r, g, b);
        int rg = rr;
        int rb = rr;

        return qRgba(rr, rg, rb, a);
    });

    return image;
}

// 复古效果(sepia)则是将红、绿、蓝三个像素，分别取这三个值的某种加权平均值，使得图像有一种古旧的效果。
QImage ImageEffects::siepa(QImage image) {
    map(&image, [](int r, int g, int b, int a) -> QRgb {
        int rr = (r * 0.393) + (g * 0.769) + (b * 0.189); // red
        int rg = (r * 0.349) + (g * 0.686) + (b * 0.168); // green
        int rb = (r * 0.272) + (g * 0.534) + (b * 0.131); // blue

        return qRgba(rr, rg, rb, a);
    });

    return image;
}

// 红色蒙版指的是，让图像呈现一种偏红的效果。算法是将红色通道设为红、绿、蓝三个值的平均值，而将绿色通道和蓝色通道都设为 0。
QImage ImageEffects::red(QImage image) {
    map(&image, [](int r, int g, int b, int a) -> QRgb {
        int rr = (r + g + b) / 3; // red
        int rg = 0; // green
        int rb = 0; // blue

        return qRgba(rr, rg, rb, a);
    });

    return image;
}

// 反转效果(invert)是指图片呈现一种色彩颠倒的效果。算法为红、绿、蓝通道都取各自的相反值(255-原值)。
QImage ImageEffects::invert(QImage image) {
    map(&image, [](int r, int g, int b, int a) -> QRgb {
        int rr = 255 - r;
        int rg = 255 - g;
        int rb = 255 - b;

        return qRgba(rr, rg, rb, a);
    });

    return image;
}

// 亮度效果(brightness)是指让图像变得更亮或更暗。算法将红色通道、绿色通道、蓝色通道，同时加上一个正值或负值。
QImage ImageEffects::brightness(QImage image, int delta) {
    map(&image, [=](int r, int g, int b, int a) -> QRgb {
        int rr = qMax(0, qMin(255, r + delta));
        int rg = qMax(0, qMin(255, g + delta));
        int rb = qMax(0, qMin(255, b + delta));

        return qRgba(rr, rg, rb, a);
    });

    return image;
}

// 把图片上每一个像素的颜色映射为函数 process() 的计算结果
void ImageEffects::map(QImage *image, std::function<QRgb (int r, int g, int b, int a)> process) {
    int w = image->width();
    int h = image->height();

    for (int y = 0; y < h; ++y) {
        for (int x = 0; x < w; ++x) {
            QRgb color = image->pixel(x, y); // 获取每个像素都要调用一次函数 pixel()，可以使用函数 scanLine() 提高效率

            int r = qRed(color);
            int g = qGreen(color);
            int b = qBlue(color);
            int a = qAlpha(color);

            color = process(r, g, b, a);
            image->setPixel(x, y, color);
        }
    }
}
```

