---
title: QSS
date: 2017-02-02 16:29:08
tags: QtBook
---
Qt 提供的 widget 的默认外观很多时候都不符合项目的界面需求，必须要改，修改一个 widget 的外观（Look and Feel）有以下的方法：

* 继承 Widget，然后在 paintEvent() 里绘制
* 继承 QStyle
* 使用 QSS（Qt Style Sheet）
* 对于 item view 来说还有一种方式，还可以使用 Delegate

这几种方式里最简单灵活的是使用 QSS，虽然有人说 QSS 的效率低，具体有多低没测试过，但是在普通 PC 上从来没感觉出来，再说现在的硬件也不差这么点性能消耗，随便一个写的差点的函数的消耗就比这多的多，作为一个实用主义者，不追求理论上的效率完美，能满足需求的前提下什么好用用什么，QSS 就是修改 widget 外观的首选，什么效果不满意，修改一下 QSS 的文件就可以看到效果，甚至不需要重新编译、打包发布程序（如果把 QSS 放在文件中，并且实现动态加载 QSS）。

**我们按下面的章节来介绍 QSS:**

* [QSS 基础](/qtbook-qss-base)
* [加载 QSS](/qtbook-qss-load)
* [盒子模型](/qtbook-qss-boxmodel) 
* [QSS 选择器](/qtbook-qss-selector)
* [Border-Image](/qtbook-qss-border-image)
* [QSS Subcontrol](/qtbook-qss-subcontrol)
* [QSS CalendarWidget](/qtbook-qss-calendar)
