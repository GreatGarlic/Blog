---
title: Qt 杂谈
date: 2016-11-08 18:51:58
tags: [QtBook, Index]
---

我们所做的事，所写的代码，都已经被其他人 * 过无数次了，在这里只不过是用我们自己的方式再演绎一次，人生如戏，全靠演技。

我不擅长于写系统的资料，所以这里很多内容看上去都比较散，更像是技术点的集合、提示、思考与总结，目标人群是有一定 Qt 基础的 Qt 爱好者。

文笔也是我的一大缺点，很是粗糙，想到哪写到哪，不成体系，甚至不同章节风格迥异，一会来个漫画，一会来个网络段子，一会神棍杂谈，宇宙洪荒大道理瞎扯蛋，目的只是用散射的思维方式从侧面启发提示相似的道理，主要是我经常天马行空的乱想如斯，实是不知如何才能表达的更贴切，希望大家多多包涵，透过现象看本质。

<!--more-->

* ...
* [绘图](/qtbook-paint/)
  * [绘图基础](/qtbook-paint-base/)
  * [绘制文本](/qtbook-paint-text/)
  * [绘制路径](/qtbook-paint-path/)
  * [贝塞尔曲线](/qtbook-paint-bezier/)
  * [线段拟合曲线](/qtbook-paint-fitting-curve)
  * [画笔](/qtbook-paint-pen/)
  * [蚂蚁线](/qtbook-paint-ant/)
  * [画刷](/qtbook-paint-brush/)
  * [渐变及原理](/qtbook-paint-gradient/)
  * [Pixmap](/qtbook-paint-pixmap/)
  * [QPainter 的状态保存与恢复](/qtbook-paint-status/)
  * [拖拽鼠标画矩形](/qtbook-paint-mouse-selection/)
  * [用画家的思维绘制图形](/qtbook-paint-artist/)
  * [绘制平滑曲线](/qtbook-paint-smooth-curve/)
  * [实时动态曲线](/qtbook-paint-realtime-curve/)
  * [使用 QChart 创建平滑曲线](/qtbook-paint-smooth-curve-qchart/)
  * [使用 QChart 显示实时动态曲线](/qtbook-paint-realtime-curve-qchart/)
  * [Clip 实现复杂绘图效果](/qtbook-paint-clip/)
  * [九宫格绘图](/qtbook-paint-nine-patch-painter)
  * [襁褓中的系统界面](/qtbook-paint-osui)
  * 仿射变换原理
  * [操作图像像素，实现各种效果](/qtbook-paint-image/)
* [Qt Style Sheet (QSS)](/qtbook-qss)
  * [QSS 基础](/qtbook-qss-base)
  * [加载 QSS](/qtbook-qss-load)
  * [盒子模型](/qtbook-qss-boxmodel)
  * [QSS 选择器](/qtbook-qss-selector)
  * [Border-Image](/qtbook-qss-border-image)
  * [QSS Subcontrol](/qtbook-qss-subcontrol)
  * [QSS CalendarWidget](/qtbook-qss-calendar)
* [访问数据库](/qtbook-db)
  * [数据库驱动](/qtbook-db-driver)
  * [访问 SQLite](/qtbook-db-sqlite)
  * [访问 MySql](/qtbook-db-mysql)
  * [数据库常用操作](/qtbook-db-common)
  * [数据库连接池](/qtbook-db-connection-pool)
  * [数据库访问工具 DBUtl](/qtbook-db-util)
* 自定义 Widget
  * [自定义 Widget 使用 QSS](/qtbook-custom-widget-enable-stylesheet)
  * [实现 Steps 路径样式](/qtbook-custom-widget-steps)
  * [按下鼠标拖动窗口](/qtbook-custom-widget-drag-to-move-window)
  * [带阴影的圆形 Label](/qtbook-custom-widget-shadow-round-label)
  * [自定义标题栏无边框阴影窗口](/qtbook-custom-widget-top-window)
  * [Layout 秘录](/qtbook-custom-widget-layout-tips)
  * [右键菜单](/qtbook-custom-widget-context-menu)
  * [QTreeView 小集](/qtbook-custom-widget-tree-view)
  * [自定义按钮组](/qtbook-custom-widget-group-buttons)
  * [异形按钮组](/qtbook-custom-widget-abnormity-buttons)
  * [分组布局](/qtbook-custom-widget-group)
  * [模型视图编程](/qtbook-custom-widget-model-view-programming)
  * [QLineEdit 中增加按钮](/qtbook-custom-widget-lineedit-with-button)
* 动画
  * QPropertyAnimation
  * 动画插值函数与高级运用
  * 传统动画实现、缓冲动画原理
  * 状态机与Qt动画一起使用
  * 与Layout一起使用时实现动画效果的展开与关闭标签页
* [多线程编程](/qtbook-thread)
  * [继承 QThread 实现多线程](/qtbook-thread-inheritance)
  * [非 UI 线程中更新 UI](/qtbook-thread-update-ui-in-nonui-thread)
  * [线程池 QThreadPool](/qtbook-thread-pool)
  * 同步
    * QMutex
    * QMutexLocker
    * QReadWriteLock
    * QSemaphore
    * QWaitCondition
    * 多种方式实现生产者消费者问题
  * 线程和 UI 的交互：汉诺塔动画实现
* Qt 的 MVC
  * Model And View
  * 动态加载数据的 Model
* [网络编程](/qtbook-network)
  * [UDP 编程](/qtbook-network-udp)
    * [单播：Unicast](/qtbook-network-udp-unicast)
    * [广播：Broadcast](/qtbook-network-udp-broadcast)
    * [组播：Multicast](/qtbook-network-udp-multicast)
  * TCP 编程
  * HTTP 编程
    * 访问 HTTP 服务
    * [访问 HTTP 服务的 HttpClient](/qtbook-network-http-httpclient)
* [单例 Singleton](/qtbook-singleton/)
  * [单例的简单实现](/qtbook-singleton-1-simple)
  * [单例的智能指针实现](/qtbook-singleton-2-auto-pointer)
  * [单例的智能指针＋宏的实现](/qtbook-singleton-3-auto-pointer-macro)
  * [单例的模版实现](/qtbook-singleton-4-template)
  * [单例的模版＋宏的实现](/qtbook-singleton-5-template-macro)
  * [单例的其他实现](/qtbook-singleton-6-other)
* 其他
  * [qmake 时复制文件](/qtbook-misc-qmake-copy-file)
  * [Single Application](/qtbook-misc-single-application)
  * [Qt5 中文乱码](/qtbook-misc-messy-code)
  * [实用正则表达式](/qtbook-misc-regex)
  * [Qt 程序简单打包](/qtbook-misc-deploy)
  * [创建使用动态链接库](/qtbook-misc-shared-library)
  * [自定义类型与 QVariant](/qtbook-misc-qvariant)
  * 枚举与 Flags


