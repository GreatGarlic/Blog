---
title: 小程序滚动
date: 2017-02-07 17:49:57
tags: FE
---
微信小程序使用 **scroll-view** 实现滚动区域，有水平滚动和垂直滚动，scroll-view 的结构如下:

```html
<scroll-view>
    <view></view>
    ...
    <view></view>
</scroll-view>
```
设置 WXSS 的时候，需要注意使用 `font-size: 0 和 white-space: nowrap`。

![](/img/fe/wx-scroll.png)
<!--more-->

```xml
<!-- 文件名: scroll.wxml -->

<view style="padding: 10rpx;">
    <view class="section">
        <view class="section__title">水平滚动</view>
        <scroll-view id="horizontal-scroll-view" scroll-x="true">
            <view class="scroll-view-item bc_green"></view>
            <view class="scroll-view-item bc_red"></view>
            <view class="scroll-view-item bc_yellow"></view>
            <view class="scroll-view-item bc_blue"></view>
        </scroll-view>
    </view>

    <view class="section">
        <view class="section__title">垂直滚动</view>
        <scroll-view id="vertical-scroll-view" scroll-y="true" bindscrolltoupper="upper" bindscrolltolower="lower" bindscroll="scroll" scroll-into-view="{{toView}}" scroll-top="{{scrollTop}}">
            <view id="green" class="scroll-view-item bc_green"></view>
            <view id="red" class="scroll-view-item bc_red"></view>
            <view id="yellow" class="scroll-view-item bc_yellow"></view>
            <view id="blue" class="scroll-view-item bc_blue"></view>
        </scroll-view>
    </view>

    <view class="btn-area">
        <button size="mini" bindtap="tap">click me to scroll into view</button>
        <button size="mini" bindtap="tapMove">click me to scroll</button>
    </view>
</view>
```

```css
<!-- 文件名: scroll.wxss -->
/* 水平滚动 */
#horizontal-scroll-view {
    height: 150px;
    white-space: nowrap; /* 很重要，否则会换行 */
}

#horizontal-scroll-view .scroll-view-item {
    width: 300rpx;
    height: 100%;
    display: inline-block;
}

/* 垂直滚动 */
#vertical-scroll-view {
    height: 150px; /* 必须设置固定的高度才能出现滚动效果 */
    font-size: 0; /* 去掉 item 之间的空白，因为有一个空格 */
}

#vertical-scroll-view .scroll-view-item {
    width: 100%;
    height: 200rpx;
    display: inline-block;
}

/* 其他 */
.section {
    margin-bottom: 8rpx;
}

button {
    width: 100%;
}

.bc_red {
    background-color: #34A7FF;
}

.bc_green {
    background-color: #2FC9E8;
}

.bc_blue {
    background-color: #41FFEC;
}

.bc_yellow {
    background-color: #2FE8A1;
}
```
水平 scroll-view 需要使用 `white-space: nowrap` 防止 view 换行。  
垂直 scroll-view 需要使用 `font-size: 0` 去掉 view 之间的空格占据的空间，并且需要给  scroll-view 设置一个固定的高度，否则看不到滚动效果。

```js
// 文件名: scroll.js

var order = ['red', 'yellow', 'blue', 'green', 'red']
Page({
    data: {
        toView: 'red',
        scrollTop: 100
    },
    upper: function(e) {
        console.log(e)
    },
    lower: function(e) {
        console.log(e)
    },
    scroll: function(e) {
        console.log(e)
    },
    tap: function(e) {
        for (var i = 0; i < order.length; ++i) {
            if (order[i] === this.data.toView) {
                this.setData({
                    toView: order[i + 1]
                })
                break
            }
        }
    },
    tapMove: function(e) {
        this.setData({
            scrollTop: this.data.scrollTop + 10
        })
    }
})
```
