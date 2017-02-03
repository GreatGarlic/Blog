---
title: QSS 基础
date: 2017-02-02 16:36:01
tags: QtBook
---
如果你会 CSS，那么 QSS 对你来说将会非常简单，QSS 的语法和 CSS 的愈发非常相似，但也有些不同，有些 CSS 的东西在 QSS 里被去掉了，QSS 也加了些自己特有的东西，不过大多数还是差不多的，下面以修改 QLabel 的样式为例，学习 QSS 的基础语法。<!--more-->

```css
QLabel {
    /* 相当于 font: bold 50px "Snell Roundhand"; */
    font-size: 50px;
    font-weight: bold;
    font-family: "Snell Roundhand";

    /* 文本的颜色 */
    color: white;

    /* 相当于 background: lightgray url(:/resources/horizontal-add-line.png); */

    background-color: lightgray;
    background-image: url(:/resources/horizontal-add-line.png);

    /* 还能使用渐变 */
    background-color: qlineargradient(x1: 0, y1: 0, x2: 0, y2: 1,
                      stop: 0 #FFFFFF, stop: 1 #BB000000);

    /* 相当于 border: 5px solid gray; */	
    border-width: 5px;
    border-color: gray;
    border-style: solid;

    /* 边框圆角 */
    border-radius: 10px;

    padding: 5px;
    margin: 10px;
}
```
下图是上面 QSS 的效果图:

![](/img/qtbook/qss/QSS-Base-1.png)

**下面具体介绍一下 QSS 的基础语法:**

* 设置字体用 `font`, 语法为：

    ```css
    font: [font-style] [font-variant] [font-weight] [font-size] [font-family]
    
    /* 按顺序设置，可以忽略其中某些值，例如：*/
    font: italic bold 12px arial, sans-serif;
    ```
    
    设置字体用 `font-style`, `font-size`, `font-weight`, `font-family`（和 CSS 一样，如果字体名字有空格则用双引号引起来，多个字体名字间用逗号分隔，如果第一个字体找不到则用第二个，第二个找不到则用第三个，依此类推）
* 设置文本颜色用 `color`
* 设置背景颜色用 `background-color`
* 设置背景使用 `background`，可以设置如下属性：
    * background-color
    * background-position
    * ~~background-size~~
    * background-repeat
    * ~~background-origin~~
    * ~~background-clip~~
    * background-attachment
    * background-image 

        ```css
        /* 可以忽略其中某些值，例如：*/
        background: lightgray url(:/resources/horizontal-add-line.png);
        ```
    
    * url 中的路径可以是：
        * 资源文件的路径（`:/` 开头）
        * 绝对路径
        * 相对于可执行文件的相对路径
* 设置背景图片用 `background-image`（先绘制背景色，然后再绘制背景图片，如果背景图片是半透明的就可以看到背景色了），默认水平和垂直重复平铺满整个 widget，同时一起设置的还可以有 `background-repeat`、`background-position`、`background-attachment`
    * background-repeat，可选的值有：
        * repeat-x：水平方向重复
        * repeat-y：垂直方向重复
        * no-repeat：不重复
    * background-position，可选的值有：
        * top left
        * top center
        * top right
        * center left
        * center center
        * center right
        * bottom left
        * bottom center
        * bottom right
        * ~~xpos ypos~~
    * background-attachment，可选的值有：
        * scroll：背景随滚动条滚动
        * fixed：背景不随滚动条滚动
* 设置背景还可以用 `border-image`，请参考 `Border-Image` 一节
* 设置边框用 `border-width`, `border-style`, `border-color`
* 设置边框用 `border`，语法为：
    
    ```css
    border: border-width border-style border-color
    
    /* 例如：*/
    border: 1px solid gray;
    ```
* 设置圆角边框用 `border-radius`，但是如果给定的半径大于对应边的一半，圆角就没有效果了，在 CSS 里没有这个问题
* 支持渐变 gradient: `qlineargradient`、`qradialgradient`、`qconicalgradient`，渐变的坐标不是用像素表示，而是把渐变的坐标的最小值定义为 0，最大值定义为 1，这种技术又叫 `Normalization`，通俗点说就是用比例表示，开始处用 0 表示，结束处用 1 表示，不管渐变的范围是 200px 还是 500px，按比例都能计算出实际的像素坐标，这样做的好处是，不需要关心渐变的像素坐标范围的具体数值。如果不用 `Normalization` 技术，widget 的大小一变化，就需要修改 QSS 里的坐标值。
    
    ```css
    /* 
    x1: 0, y1: 0，渐变的开始位置，为 border rectangle 的左上角（请参考盒子模型）
    x2: 0, y2: 1，渐变的开始位置，为 border rectangle 的左下角
    stop: 0.1 #FF0000，在 0.0 处渐变的颜色为 #FF0000
    stop: 0.6 #00FF00，在 0.6 处渐变的颜色为 #00FF00
    stop: 1.0 #0000FF，在 1.0 处渐变的颜色为 #0000FF
    */
    background-color: qlineargradient(x1: 0, y1: 0, x2: 0, y2: 1,
                            stop: 0.1 #FF0000, 
                            stop: 0.6 #00FF00,
                            stop: 1.0 #0000FF);
    ```
    渐变效果比较复杂，可以使用 Qt Desginer 的 QSS 编辑器来帮助我们可视化的实现复杂的渐变效果，如下图:

    ![](/img/qtbook/qss/QSS-Base-2.png)
    
* `Padding` 和 `margin` 参考 `盒子模型` 一节
* 设置图标，如 QToolButton 的图标：
    
    ```css
    /* 图标大小 */
    icon-size: 20px 20px;
    
    /* 图标文件 */
    qproperty-icon: url(:/resources/tabset-left.png);
    ```
* 设置宽度用 `width`，高度用 `height`，设置 subcontrol 的时候比较有用
* 设置最小宽度用 `min-width`，最小高度用 `min-height`（是 content rectangle 的宽和高）
* 设置最大宽度用 `max-width`，最大高度用 `max-height`
* 遗憾的是，QSS 不支持阴影

上面的 QSS 的虽然只是基础，但是很重要，大多数的时候都要用到它们，用来修改 QLabel，QPushButton，QFrame，QWidget 等的样式还基本够用了，不过如果要修改复杂一点的 widget 的样式，如 QSpinBox，QScrollBar 等，上面的知识是不够的，要想掌握好 QSS，还必须了解 `QSS 的选择器`，`盒子模型`，`Border-Image`，`Subcontrol` 等，这些在后面都有专门的章节介绍。
